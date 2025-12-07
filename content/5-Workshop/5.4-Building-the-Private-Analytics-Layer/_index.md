---
title: "Building the Private Analytics Layer"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

This section explains in detail how the **private analytics layer** is implemented so that batch ETL can run **entirely inside private subnets**, without exposing internal components to the public Internet.

The private analytics layer consists of:

- A **VPC-enabled ETL Lambda function** running in a **private subnet**.  
- An **S3 Gateway VPC Endpoint** that allows the ETL Lambda to access S3 over the **AWS private network**.  
- A **PostgreSQL Data Warehouse** running on an **EC2 instance** in a separate private subnet.  

Together, these components form the backbone of the batch-processing pipeline:

> S3 Raw Clickstream bucket → ETL Lambda (in VPC) → PostgreSQL Data Warehouse (EC2, private subnet)

---

### Networking design for the analytics layer

The VPC is logically divided into subnets with distinct roles:

- **Public Subnet – OLTP (10.0.1.0/24)**  
  - Hosts the OLTP PostgreSQL EC2 instance.  
  - Has a route `0.0.0.0/0 → Internet Gateway (IGW)` for inbound/outbound Internet access.  

- **Private Subnet – Analytics (10.0.2.0/24)**  
  - Hosts the **Data Warehouse EC2** instance and the **R Shiny Server**.  
  - Has **no route to the Internet Gateway** and **no NAT Gateway**.  
  - Only local VPC routes plus internal connectivity to other private subnets.

- **Private Subnet – ETL (10.0.3.0/24)**  
  - Hosts the **VPC-enabled ETL Lambda** and the **S3 Gateway VPC Endpoint**.  
  - Also has **no 0.0.0.0/0 route**, and no NAT Gateway.  
  - Can reach S3 privately through the Gateway Endpoint.

This design ensures that:

- The Data Warehouse and R Shiny are **not directly reachable from the public Internet**.  
- The ETL Lambda can access S3 and the Data Warehouse **only over private AWS networking**.  
- There is **no NAT Gateway**, which reduces cost and simplifies the network footprint.

---

### Creating and configuring the S3 Gateway VPC Endpoint

The S3 Gateway VPC Endpoint allows resources in private subnets to reach S3 **without using public IPs**.

**Figure 5-9: S3 Gateway VPC Endpoint attached to private route tables**

The screenshot shows the Gateway VPC Endpoint for the Amazon S3 service (`com.amazonaws.ap-southeast-1.s3`) in the workshop VPC.  
The endpoint is in the `Available` state and is associated with the two private route tables, which are in turn attached to the Analytics and ETL subnets.  
This configuration ensures that traffic from the ETL Lambda and the Data Warehouse instance to S3 stays inside the AWS network and does not require a NAT Gateway.

![Figure 5-9: Gateway VPC Endpoint attached to private route tables](/images/5-4-s3-gateway-vpce.png)

#### Step 1 – Create the Gateway Endpoint

1. Open the **VPC Console** in the AWS Management Console.  
2. Navigate to **Endpoints** → click **Create endpoint**.  
3. Under **Service category**, select **AWS services**.  
4. In the search box, type `s3` and select the entry:

   ```text
   com.amazonaws.<region>.s3
   ```

5. For **Endpoint type**, select **Gateway**.  
6. For **VPC**, choose the project VPC that contains your analytics and ETL subnets.  
7. Under **Route tables**, select the route table associated with the **ETL private subnet (10.0.3.0/24)** and, optionally, the **Analytics private subnet (10.0.2.0/24)** if you also want the DW EC2 instance to access S3 privately.  
8. In the **Policy** section, you can start with **Full access** for the workshop:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:*",
         "Resource": "*"
       }
     ]
   }
   ```

   Later, this can be restricted to a specific bucket or prefix.

9. Click **Create endpoint**.

#### Step 2 – Verify route table entries

After the endpoint is created, AWS automatically adds routes to the selected route tables.

1. In the **VPC Console**, open **Route tables**.  
2. Select the route table used by the **ETL private subnet**.  
3. On the **Routes** tab, verify that:

   - There is a local route:

     ```text
     10.0.0.0/16 → local
     ```

   - There is a prefix-list route to S3​, pointing to the new endpoint:

     ```text
     pl-xxxxxxxx → vpce-xxxxxxxx  (Gateway Endpoint to S3)
     ```

   - There is **no** route:

     ```text
     0.0.0.0/0 → igw-xxxxxxx
     ```

   - There is **no** NAT Gateway target.

This confirms that the private subnets do **not** send traffic directly to the Internet, but **can reach S3** through the Gateway Endpoint.

---

### Configuring the ETL Lambda inside the VPC

The ETL Lambda function must be placed inside the VPC so it can:

- Reach S3 via the **Gateway Endpoint**.  
- Connect to the **PostgreSQL Data Warehouse** in the Analytics private subnet.

#### Step 1 – Attach the Lambda to the VPC

1. Open the **Lambda Console** and choose the ETL function, for example:

   ```text
   clickstream-etl-lambda
   ```

2. Go to the **Configuration** tab → **Network** (or **Environment → VPC** depending on the UI).  
3. Click **Edit** and set:

   - **VPC**: your project VPC.  
   - **Subnets**: select the **ETL private subnet (10.0.3.0/24)** (you can select multiple subnets in the same AZ or different AZs for resilience).  
   - **Security groups**: choose a security group that:
     - Allows outbound traffic to S3 (via the Gateway Endpoint).  
     - Allows outbound traffic to the Data Warehouse EC2 instance (port 5432).

4. Save the configuration. Lambda will now create ENIs (elastic network interfaces) in the selected subnet(s) the next time it runs.

#### Step 2 – IAM role permissions for ETL Lambda

The ETL Lambda **execution role** must include:

- Permission to **read from** the Raw Clickstream S3 bucket, e.g.:

  ```json
  {
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::clickstream-raw-<account>-<region>",
      "arn:aws:s3:::clickstream-raw-<account>-<region>/events/*"
    ]
  }
  ```

- Permission to **write logs** to CloudWatch Logs (usually added by default).  
- No direct permission to manage infrastructure (keep the role minimal).

#### Step 3 – Database connectivity configuration

In the ETL Lambda environment variables, store:

- `DW_HOST` – the private IP or hostname of the Data Warehouse EC2 instance.  
- `DW_PORT` – typically `5432`.  
- `DW_USER` / `DW_PASSWORD` – credentials with sufficient privileges to insert into DW tables.  
- `DW_DATABASE` – the name of the analytics database.

The Lambda code will use these environment variables to establish a connection (for example, using a PostgreSQL client library).

---

### Implementing the ETL logic

The ETL Lambda performs the following high-level steps each time it runs (either on a schedule via EventBridge or triggered manually):

1. **Determine the time window / S3 prefix to process**  
   - For example, all objects under `events/YYYY/MM/DD/HH/` for the last hour.

2. **List relevant S3 objects**  
   - Use `ListObjectsV2` on the Raw Clickstream bucket with the chosen prefix.

3. **Read and parse events**  
   - For each object, call `GetObject` and parse the JSON content.  
   - Validate important fields (event name, timestamp, user/session IDs, product IDs).

4. **Transform to analytical schema**  
   - Map raw JSON to relational tables, for example:

     - `dim_users` – user-level attributes.  
     - `dim_products` – product attributes.  
     - `fact_events` – individual events with foreign keys to users and products.  
     - `fact_sessions` – aggregated session-level metrics.

   - Apply simple business rules, such as:
     - Derive a **date** and **hour** dimension from timestamps.  
     - Normalize event names.  
     - Filter out test or malformed events.

5. **Insert into the Data Warehouse**

   - Open a transaction against the PostgreSQL Data Warehouse.  
   - Use batched `INSERT` statements or `COPY`-like bulk inserts (depending on the implementation).  
   - Commit if everything is successful; roll back on errors.

6. **Mark progress (optional)**  
   - Store a processing marker (e.g., last processed hour) in a control table or a metadata S3 prefix to avoid re-processing.

Example of a simple target fact table for events:

```sql
CREATE TABLE IF NOT EXISTS fact_events (
    event_id          UUID PRIMARY KEY,
    event_timestamp   TIMESTAMPTZ NOT NULL,
    event_name        TEXT NOT NULL,
    user_id           TEXT,
    session_id        TEXT,
    client_id         TEXT,
    product_id        TEXT,
    page_url          TEXT,
    traffic_source    TEXT,
    created_at        TIMESTAMPTZ DEFAULT NOW()
);
```

The ETL Lambda will insert data into `fact_events` based on the parsed S3 JSON files.

**Figure 5-10: VPC configuration of the ETL Lambda function**

The screenshot shows the `SBW_Lamda_ETL` function attached to the `SBW_Project-vpc`.  
The Lambda uses two private subnets (one in each Availability Zone) and a dedicated security group (`sg_Lamda_ETL`) that controls all network access.  
This configuration allows the function to reach the S3 Gateway VPC Endpoint and the Data Warehouse instance without exposing the Lambda to the public internet.

![Figure 5-10: VPC configuration of the ETL Lambda function](/images/5-4-lambda-etl-vpc.png)

---

### Security groups and connectivity

To ensure that only the necessary traffic is allowed:

- **Security Group for the Data Warehouse EC2 (SG-DW)**

  - Inbound rules:
    - Allow `5432/tcp` **from the ETL Lambda security group** (not from `0.0.0.0/0`).  
  - Outbound rules:
    - Allow all outbound traffic (or restrict as needed for updates/monitoring).

- **Security Group for the ETL Lambda (SG-ETL)**

  - Inbound rules:
    - Not needed (Lambda does not accept inbound connections).  
  - Outbound rules:
    - Allow traffic to:
      - The Data Warehouse EC2 private IP on port `5432`.  
      - The S3 Gateway VPC Endpoint (this is usually covered by allowing all outbound traffic within the VPC).

This setup ensures that:

- Only the ETL Lambda can talk to the Data Warehouse (no direct access from the public Internet).  
- The Data Warehouse EC2 does not accept arbitrary incoming connections.

---

### Testing the ETL pipeline from end to end

To validate that the private analytics layer is working correctly:

1. **Generate fresh events**  
   - Follow Section 5.3 to generate new clickstream events (page views, product views, add-to-cart, checkout).

2. **Trigger the ETL Lambda**

   - Option A: Wait for the scheduled **EventBridge rule** (e.g., every 30 minutes).  
   - Option B: In the EventBridge console, select the ETL rule (for example `clickstream-etl-schedule`) and choose **Run now**.  
   - Option C: In the Lambda console, use the **Test** button with a minimal test event to manually invoke the ETL.

3. **Check CloudWatch Logs**

   - Open the ETL Lambda log group in **CloudWatch Logs**.  
   - Verify that:
     - The Lambda successfully listed and read S3 objects.  
     - It processed the expected number of events.  
     - It connected to the Data Warehouse and executed inserts without errors.

4. **Verify data in the Data Warehouse**

   - Connect to the Data Warehouse EC2 instance using **Session Manager** or SSH.  
   - From there, use `psql` or a SQL client to run queries such as:

     ```sql
     SELECT event_name, COUNT(*) AS total_events
     FROM fact_events
     GROUP BY event_name
     ORDER BY total_events DESC;
     ```

     ```sql
     SELECT product_id, COUNT(*) AS view_count
     FROM fact_events
     WHERE event_name = 'product_view'
     GROUP BY product_id
     ORDER BY view_count DESC
     LIMIT 10;
     ```

   - Confirm that the counts and top products roughly match the behaviour you generated on the website.

---

### Summary

By the end of this section, you will have:

- Configured an **S3 Gateway VPC Endpoint** so that private subnets can access S3 without a NAT Gateway.  
- Attached the **ETL Lambda** to the VPC and ensured it can reach both S3 and the Data Warehouse over private networking.  
- Implemented ETL logic that reads raw JSON clickstream files, transforms them, and loads them into a **PostgreSQL Data Warehouse** in a private subnet.  
- Verified end-to-end connectivity and data flow by inspecting logs and running SQL queries.

In the next section (5.5), you will use **R Shiny dashboards** running on the Data Warehouse EC2 instance to visualize the processed data and build interactive analytics views.

---

**Figure 5-11: Private analytics layer with ETL Lambda, Data Warehouse, and S3 Gateway Endpoint**

The diagram shows how the batch ETL pipeline runs entirely inside the Analytics private subnet:

- **EventBridge** (bottom right) triggers the **ETL Lambda** on a schedule.  
- The ETL Lambda reads raw clickstream files from the **S3 Raw Clickstream bucket** and reaches S3 through the **S3 Gateway VPC Endpoint** (no Internet/NAT required).  
- The ETL Lambda writes transformed data into the **PostgreSQL Data Warehouse EC2 instance**, and the **R Shiny Server** running on the same instance reads from the Data Warehouse to power analytics dashboards.

![Figure 5-11: Private analytics layer with ETL Lambda, Data Warehouse, and S3 Gateway Endpoint](/images/5-4-etl-s3-endpoint.png)