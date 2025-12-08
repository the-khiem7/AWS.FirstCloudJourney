---
title: "Visualizing Analytics with Shiny Dashboards"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

In the previous sections, you ingested clickstream events into the **S3 Raw Clickstream bucket** and used a **VPC-enabled ETL Lambda** to load curated data into the **PostgreSQL Data Warehouse** in a private subnet.

This section focuses on the **final mile** of the pipeline:

- Validating that the ETL job has populated the Data Warehouse with meaningful data.  
- Using **R Shiny dashboards** running on the same EC2 instance as the Data Warehouse to explore user behaviour, funnels, and product performance.

The goal is not only to “see some charts”, but to **connect each visualization back to the underlying data model and business questions**.

---

### Triggering batch ETL and verifying data in the Data Warehouse

Before looking at dashboards, make sure that the Data Warehouse contains fresh data.

#### Step 1 – Generate new clickstream events

1. Open the CloudFront domain of the e-commerce application, for example:

   ```text
   https://dxxxxxxxx.cloudfront.net
   ```

2. Sign in with a test user via **Amazon Cognito**.  
3. Perform a realistic browsing session, such as:

   - Visit the home page and at least one category page.  
   - Search for a product or filter by brand/category.  
   - Open three or more product detail pages.  
   - Add one or two items to the cart.  
   - Remove an item, change quantity, and proceed to the checkout flow.  
   - Optionally, complete an order so that purchase events are generated.

4. Wait 1–2 minutes to ensure all events have been sent to **API Gateway → Lambda Ingest → S3 Raw Clickstream bucket**.

#### Step 2 – Run the ETL job

You have three main options to run the ETL Lambda:

- **A. Wait for the EventBridge schedule**  
  - If the ETL rule is configured to run every 15 or 30 minutes, you can simply wait for the next trigger.

- **B. Manually trigger via EventBridge**  
  1. Open **Amazon EventBridge** console.  
  2. Go to **Rules** and select the ETL rule, for example:

     ```text
     clickstream-etl-schedule
     ```

  3. Choose **Actions → Run now** to execute the rule immediately.

- **C. Manually trigger via Lambda console**  
  1. Open the **Lambda Console** and select the ETL function:

     ```text
     clickstream-etl-lambda
     ```

  2. On the **Test** tab, create or reuse a test event (payload `{}` is sufficient if your code ignores the input).  
  3. Click **Test** to invoke the ETL Lambda.

#### Step 3 – Check ETL execution in CloudWatch Logs

1. Open **CloudWatch Logs** in the AWS console.  
2. Navigate to the log group for the ETL Lambda function.  
3. Open the most recent log stream and check for entries such as:

   - “Listing S3 objects under prefix `events/YYYY/MM/DD/HH/` …”  
   - “Read N files, parsed M events.”  
   - “Inserted K rows into `fact_events`, L rows into `dim_products`, etc.”  
   - Any error messages or stack traces (if present).

If there are errors, review:

- IAM permissions for S3 and the database.  
- VPC configuration (subnets, route tables, Gateway Endpoint).  
- Database connectivity (host, port, credentials).

**Figure 5-12: CloudWatch logs for the latest ETL run**

The screenshot shows the CloudWatch log stream for a recent execution of the ETL Lambda function.  
The `INIT_START`, `START`, `END`, and `REPORT` entries confirm that the function executed successfully, and the reported duration and memory usage are within the expected range.  
Checking these logs helps ensure that the Data Warehouse contains fresh data before exploring the Shiny dashboards.

![Figure 5-12: CloudWatch logs for the latest ETL run](/images/5-5-etl-cloudwatch-logs.png)

#### Step 4 – Validate data with SQL queries

Once the ETL reports success, verify that the Data Warehouse has been updated.

1. Connect to the **Data Warehouse EC2 instance** using **AWS Systems Manager Session Manager** (no SSH keys or bastion hosts required).  
2. From within the Session Manager shell, connect to PostgreSQL DW using `psql` or a graphical client:

Run basic checks like:

```sql
-- 1. How many events are in the fact table?
SELECT COUNT(*) AS total_events
FROM fact_events;
```

```sql
-- 2. Events by type (page views, product views, add-to-cart, etc.)
SELECT event_name, COUNT(*) AS total_events
FROM fact_events
GROUP BY event_name
ORDER BY total_events DESC;
```

```sql
-- 3. Top 10 most viewed products
SELECT
    product_id,
    product_name,
    COUNT(*) AS view_count
FROM fact_events
WHERE event_name = 'product_view'
GROUP BY product_id, product_name
ORDER BY view_count DESC
LIMIT 10;
```

Optional deeper checks:

```sql
-- 4. Funnel-like view: from product view to add-to-cart
SELECT
    product_id,
    SUM(CASE WHEN event_name = 'product_view'  THEN 1 ELSE 0 END) AS product_views,
    SUM(CASE WHEN event_name = 'add_to_cart'   THEN 1 ELSE 0 END) AS add_to_cart_events
FROM fact_events
GROUP BY product_id
ORDER BY product_views DESC
LIMIT 10;
```

Compare these numbers with your test session:

- Did you view at least as many products as the query shows?  
- Do the products you interacted with appear near the top of the list?  
- If you completed a purchase, can you see related events in the data?

---

### Accessing the Shiny dashboards (from a private EC2 instance)

The **R Shiny Server** runs on the same private EC2 instance as the Data Warehouse, without a public IP. To access it securely, use **AWS Systems Manager Session Manager port forwarding**—no SSH keys or bastion hosts required.

#### Accessing Shiny via Session Manager Port Forwarding

1. Ensure the **Session Manager plugin** for the AWS CLI is installed on your local machine.  
   - Installation instructions: [AWS Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

2. Run the following command to forward the Shiny port (default 3838) to your local machine:

   ```bash
   aws ssm start-session \
       --target <instance-id> \
       --document-name AWS-StartPortForwardingSession \
       --parameters '{"portNumber":["3838"],"localPortNumber":["8080"]}'
   ```

   Replace `<instance-id>` with the EC2 instance ID of your Data Warehouse instance.

3. Keep this terminal session open. You should see a message like:

   ```text
   Starting session with SessionId: ...
   Port 8080 opened for sessionId ...
   ```

4. Open a browser on your local machine and navigate to:

   ```text
   http://localhost:8080/
   ```

   or to the specific Shiny app, for example:

   ```text
   http://localhost:8080/clickstream-analytics
   ```

#### Benefits of Session Manager Port Forwarding

- **No SSH exposure**: The private EC2 instance does not need inbound SSH rules or public IP addresses.  
- **No bastion hosts**: Eliminates the need to manage and secure jump servers.  
- **IAM-based access control**: Permissions are managed through IAM policies, with full audit trails in CloudTrail.  
- **Encrypted tunnels**: All traffic is encrypted over HTTPS, staying within the AWS network via VPC Interface Endpoints.

---

### Exploring the dashboards

Once you have access to the Shiny homepage or specific app, you should see one or more dashboards built on top of the Data Warehouse.

Typical views might include:

#### Funnel / User Journey Dashboard

Shows how users flow through key steps such as:

1. `page_view` → 2. `product_view` → 3. `add_to_cart` → 4. `checkout_start` → 5. `purchase`

Common visualizations:

- A funnel chart with counts at each step.  
- Drop-off percentages between one step and the next.  
- Filters for date ranges, device type, or traffic source.

Questions you can answer:

- How many users start at a product view and reach the checkout step?  
- Where do most users abandon the journey (e.g., before add-to-cart, or during checkout)?  
- Does the funnel performance change over time or by traffic source?

#### Product Performance Dashboard

Focuses on product-level metrics:

- Most viewed products (`product_view` events).  
- Products with the highest add-to-cart or purchase events.  
- Conversion rate per product (add-to-cart / product views, purchases / product views).

Possible visualizations:

- Bar charts ranking products by views or purchases.  
- Tables showing product name, category, and key engagement metrics.  
- Filters by category, brand, or price range.

Questions you can answer:

- Which products attract the most attention but have low conversion?  
- Which categories or brands perform best overall?  
- Are there products that rarely get views and might need promotion?

#### Time Series / Activity Over Time

Shows how user activity changes over time:

- Number of events per hour or per day.  
- Separate lines for page views, product views, add-to-cart, purchases.  
- Optional breakdown by device type or traffic source.

Questions you can answer:

- What times of day see the highest browsing activity?  
- Are there specific days of the week with better conversion?  
- Do special campaigns or promotions (if recorded) align with spikes in activity?

---

### Relating dashboards back to the Data Warehouse schema

Each dashboard is powered by SQL queries against the Data Warehouse tables, such as:

- `fact_events` – granular event-level data.  
- `dim_products` – product attributes (name, category, brand, etc.).  
- `dim_users` – user attributes (registered vs. guest, segment, etc.).  
- `fact_sessions` or `fact_funnels` – pre-aggregated session or funnel metrics (if modeled).

When you interact with filters in Shiny (e.g., selecting a date range, category, or event type), the Shiny app typically:

1. Builds a SQL query using the selected parameters.  
2. Sends the query to PostgreSQL.  
3. Receives the aggregated results.  
4. Renders them as charts or tables in the browser.

As an exercise, you can:

- Open the Shiny app source code (R scripts) on the EC2 instance.  
- Locate the SQL queries used for each widget.  
- Compare those queries with the **manual SQL** you ran earlier in this section.

This helps build confidence that:

- The dashboards are consistent with the underlying data.  
- You can reproduce key numbers directly using SQL if needed.

---

### Summary

By completing this section, you have:

- Ensured that the **ETL batch job** has successfully populated the PostgreSQL Data Warehouse with fresh clickstream data.  
- Validated the data using direct **SQL queries** (event counts, product views, basic funnel metrics).  
- Accessed **R Shiny dashboards** running on the private EC2 instance via secure port forwarding.  
- Explored key dashboards for user journeys, product performance, and activity over time.  
- Connected Shiny visualizations back to the underlying Data Warehouse schema and queries.

In the next section (**5.6 Summary & Clean up**), you will briefly recap the main learnings from the workshop and review which AWS resources should be stopped or deleted to avoid unnecessary costs.

---

**Figure 5-13: Shiny dashboard for clickstream analytics**

The screenshot shows a Shiny dashboard built on top of the PostgreSQL Data Warehouse:

- A user journey funnel from product views to purchases.  
- A time-series chart of events per day (page views, product views, add-to-cart, purchases).  
- A top products table listing the items with the highest engagement and conversion.  
- Filters at the top (date range, event type, device, traffic source) to slice and dice the analysis.

![Figure 5-13: Shiny dashboard for clickstream analytics](/images/5-5-shiny-dashboard.png)
