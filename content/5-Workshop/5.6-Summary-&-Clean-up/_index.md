---
title: "Summary & Clean up"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

This final section:

- Recaps the key concepts and components you worked with throughout the workshop.  
- Highlights how they fit together into a cohesive **Batch-based Clickstream Analytics Platform**.  
- Provides a **clean-up checklist** so you can safely remove or stop AWS resources and avoid unnecessary costs once you finish experimenting.

---

### Summary of key learnings

Across Sections **5.1–5.5**, you have built and validated a complete end-to-end analytics pipeline for an e-commerce website selling computer products.

#### Architecture and design

From **5.1 Objectives & Scope** and **5.2 Architecture Walkthrough**, you learned to:

- Describe the core building blocks of the platform:

  - **Frontend & user-facing domain**:  
    - Next.js application on **AWS Amplify Hosting**.  
    - Delivered via **Amazon CloudFront**.  
    - User authentication through **Amazon Cognito**.

  - **Ingestion & data lake domain**:  
    - **Amazon API Gateway (HTTP API)** as the clickstream ingestion endpoint.  
    - **Lambda Ingest** function validating and enriching events.  
    - **S3 Raw Clickstream bucket** storing time-partitioned JSON logs.

  - **Analytics & Data Warehouse domain**:  
    - **VPC** with a **public OLTP subnet**, and **private Analytics + ETL subnets**.  
    - **PostgreSQL Data Warehouse on EC2** in the Analytics private subnet.  
    - **R Shiny Server** co-located on the same EC2 instance for analytics dashboards.  
    - **VPC-enabled ETL Lambda** in the ETL private subnet.  
    - **AWS Systems Manager Session Manager** with VPC Interface Endpoints for secure, zero-SSH admin access to private instances.

- Explain why the platform separates:

  - **OLTP (Operational)** workloads (live orders, inventory, users) and  
  - **Analytics (Read-heavy)** workloads (funnel analysis, product performance, time series).

This separation improves performance, resilience, and security.

##### Ingestion path: Frontend → API Gateway → S3

From **5.3 Implementing Clickstream Ingestion**, you:

- Implemented and/or understood a **tracking flow** where:

  - The browser captures page views, product views, add-to-cart, and checkout events.  
  - Events are serialized into JSON payloads containing user, session, and product metadata.  
  - The frontend calls `POST /clickstream` on **API Gateway**.

- Saw how **Lambda Ingest**:

  - Parses incoming events.  
  - Enforces minimal validation and adds server-side metadata.  
  - Writes batched JSON objects into the **S3 Raw Clickstream bucket** using a partitioned prefix such as:

    ```text
    events/YYYY/MM/DD/HH/events-<uuid>.json
    ```

- Practiced **manual testing** using both:

  - The real frontend (via the browser developer tools), and  
  - API clients such as Postman or Thunder Client.

#### Private analytics layer: Gateway Endpoint, ETL Lambda, Data Warehouse

From **5.4 Building the Private Analytics Layer**, you:

- Configured a **Gateway VPC Endpoint for S3**, ensuring that:

  - Private subnets **do not** require public IPs or a NAT Gateway to reach S3.  
  - Traffic between ETL Lambda and S3 stays on the **AWS private network**.

- Attached the **ETL Lambda** to the VPC by:

  - Selecting the ETL private subnet and appropriate security groups.  
  - Giving the Lambda execution role minimal IAM permissions to read from the Raw bucket and write to CloudWatch Logs.

- Implemented ETL logic that:

  - Lists relevant S3 objects for a given time window.  
  - Parses raw JSON clickstream events.  
  - Transforms them into relational structures (e.g., `fact_events`, `dim_products`).  
  - Loads them into a **PostgreSQL Data Warehouse** on an EC2 instance in the Analytics private subnet.

- Verified connectivity and security through:

  - Security groups allowing **only the ETL Lambda** to connect to the DW on port `5432`.  
  - Route tables with local + prefix-list routes, and **no 0.0.0.0/0** in private subnets.

#### Analytics and visualization with Shiny

From **5.5 Visualizing Analytics with Shiny Dashboards**, you:

- Triggered the ETL job (via EventBridge or manually) and validated data freshness using SQL queries:

  - Event counts.  
  - Distribution of event types.  
  - Top viewed products.  
  - Simple funnel-like metrics.

- Accessed **R Shiny dashboards** running on the same private EC2 instance as the Data Warehouse using **AWS Systems Manager Session Manager port forwarding**:

  - No SSH keys or bastion hosts required.  
  - Fully encrypted tunnels over HTTPS via VPC Interface Endpoints.  
  - IAM-based access control with full audit trails.

- Explored dashboards that represent:

  - **Funnels and user journeys** (page view → product view → add-to-cart → checkout → purchase).  
  - **Product performance** (top products by views and purchases).  
  - **Time-series trends** (events per hour/day).

- Connected Shiny visualizations back to underlying **SQL queries and DW tables**, confirming that:

  - The dashboards are consistent with the data.  
  - Key business metrics can be verified manually when needed.

---

### Clean-up checklist (AWS resources)

After completing the workshop, it is recommended to **clean up resources** to avoid charges. The exact actions depend on whether this environment is:

- A short-lived lab or demo environment, or  
- A persistent development/staging environment.

> **Important:** Before deleting anything, confirm that **no one else** is using the same AWS account or resources for ongoing work.

#### EC2 instances

1. **OLTP EC2 instance (Public Subnet)**

   - If you no longer need the operational database:
     - Stop the instance to pause compute charges, or  
     - Terminate it to permanently delete it.  
   - Before termination, consider:
     - Taking a final backup or snapshot of the volume.  
     - Exporting a logical dump (e.g., `pg_dump`) of important data.

2. **Data Warehouse + Shiny EC2 instance (Private Subnet)**

   - If you only needed this instance for the workshop:
     - Stop or terminate it after confirming that you have exported any necessary analytical results or schema definitions.  
   - If you plan to extend the analytics layer later:
     - You may keep the instance, but ensure that:
       - Unused services (e.g., experimental Shiny apps) are removed.  
       - Security groups remain restrictive (no broad inbound rules).

#### Lambda functions and EventBridge rules

1. **Lambda Ingest function**

   - If you will not send more clickstream events:
     - Consider deleting the function to avoid confusion in the console.  
   - Otherwise, you can keep it for future experimentation.

2. **ETL Lambda function**

   - If the Data Warehouse EC2 instance is stopped or terminated:
     - Disable or delete the ETL Lambda (and its EventBridge rule) to avoid failing invocations.  

3. **EventBridge ETL schedule**

   - Navigate to **Amazon EventBridge → Rules**.  
   - Disable or delete the scheduling rule (for example `clickstream-etl-schedule`) so it no longer triggers ETL runs.

#### S3 buckets and data

1. **Raw Clickstream S3 bucket**

   - Decide whether the raw clickstream data should be retained:

     - For **short-lived labs**, you can safely delete the `events/YYYY/MM/DD/HH/` prefixes created during testing.  
     - For **ongoing projects**, keep the bucket but:
       - Enable S3 **lifecycle policies** if needed (e.g., transition older data to cheaper storage classes, or delete after N days).

2. **Other project-specific buckets**

   - If you created additional buckets specifically for this workshop (for logs, artifacts, or screenshots), review and delete them if they are no longer required.

> **Note:** S3 storage costs can accumulate over time; periodically review bucket contents and lifecycle rules.

#### VPC Endpoints and networking components

1. **S3 Gateway VPC Endpoint**

   - If the VPC will no longer be used for analytics:
     - You may delete the **Gateway Endpoint**.  
   - If the VPC remains active and other workloads may benefit from private S3 access:
     - It can be left in place.

2. **SSM Interface VPC Endpoints**

   - Delete the VPC Interface Endpoints for AWS Systems Manager if no longer needed:
     - `com.amazonaws.<region>.ssm`  
     - `com.amazonaws.<region>.ssmmessages`  
     - `com.amazonaws.<region>.ec2messages`  
   - Navigate to **VPC** → **Endpoints**, select each endpoint, and choose **Actions** → **Delete endpoint**.

   > **Note**: Interface Endpoints incur hourly charges (~$0.01/hour per endpoint per AZ). Gateway Endpoints for S3 are free.

3. **Route tables, subnets, and VPC**

   - For a dedicated lab VPC:
     - Consider deleting the entire VPC (which will also remove associated subnets, route tables, and endpoints) **only after** ensuring no active EC2, RDS, or other critical resources depend on it.  
   - For a shared VPC:
     - Clean up only the subnets and route tables that were created exclusively for this workshop, taking care not to disrupt other environments.

#### CloudWatch Logs and monitoring

1. **CloudWatch Logs for Lambda and API Gateway**

   - Log groups can grow in size over time.  
   - Set an appropriate **retention policy** (for example, 7, 30, or 90 days) for:
     - Lambda Ingest logs.  
     - ETL Lambda logs.  
     - API Gateway access logs.

2. **CloudWatch Alarms and metrics**

   - If you created alarms specific to this environment (e.g., error count alarms for ETL or ingestion):
     - Disable or delete them if the environment is being decommissioned.

**Figure 5-14: CloudWatch alarms for the clickstream platform**

The screenshot shows a set of CloudWatch alarms for the ingestion path and the Data Warehouse, including:

- API latency and 4xx/5xx rate alarms.  
- Lambda ingest duration and throttling alarms.  
- EC2 Data Warehouse status check and CPU utilization alarms.

Most alarms are in `OK` or `Insufficient data` state, indicating that the system is currently healthy and ready to be stopped or scaled down after the workshop.

![Figure 5-14: CloudWatch alarms for the clickstream platform](/images/5-6-cloudwatch-alarms.png)

#### IAM roles and policies

1. **Lambda execution roles**

   - Review IAM roles created for:
     - Lambda Ingest.  
     - ETL Lambda.  

   - If they are no longer needed:
     - Detach any inline or attached policies.  
     - Delete the roles to avoid accumulating unused IAM identities.

2. **Test users and permissions**

   - If you created dedicated IAM users or test roles for the workshop, consider removing them or tightening their permissions.

#### Amplify, Cognito, and API Gateway

1. **Amplify app**

   - If the Next.js frontend was deployed only for this workshop and will not be used again:
     - Remove the **Amplify app** and associated branches/environments.  

2. **Cognito User Pool**

   - Remove test users or delete the entire User Pool if it was dedicated to this environment.

3. **API Gateway HTTP API**

   - Delete the clickstream ingestion API if you no longer need it, especially if it was created only for the lab.

---

### What to keep for future work

If you plan to **extend** this project beyond the workshop, consider keeping:

- **The code repositories**:

  - Next.js frontend (including tracking logic).  
  - Lambda functions (Ingest and ETL).  
  - Shiny app code.  
  - Infrastructure-as-Code (Terraform or CloudFormation templates).

- **The Data Warehouse schema and a small sample dataset**:

  - A minimal EC2 instance and database with a sample of events can be enough to continue developing dashboards and queries without ingesting large volumes of new data.

- **Architecture diagrams and notes**:

  - Diagrams used as Figures 5-1 to 5-6.  
  - Notes on design decisions (e.g., using Gateway Endpoint instead of NAT Gateway).

These assets make it easier to resume the project later or adapt it into a more advanced solution (for example, migrating to **Amazon Redshift Serverless** or adding **real-time streaming**).

---

### Closing remarks

This workshop demonstrated how to:

- Design and implement a **secure, cost-conscious, and extensible** analytics architecture on AWS.  
- Collect clickstream events from a real e-commerce frontend.  
- Build a private ETL process that reads from S3 and loads into a dedicated Data Warehouse.  
- Visualize user behaviour and product performance through R Shiny dashboards, without exposing the analytics backend to the public Internet.

You now have a working reference for building similar analytics platforms and a strong foundation for future enhancements such as:

- Real-time streams (Kinesis / Kafka).  
- More advanced attribution and user segmentation models.  
- Migrating the Data Warehouse to managed services like Amazon Redshift.

---

**Figure 5-15: Summary of resources to check after the workshop**

The diagram or table for this figure should list, for each AWS service:

- The resource type (EC2, S3, Lambda, EventBridge, API Gateway, VPC Endpoint, etc.).  
- Example resource names.  
- Recommended action after the workshop (Keep / Stop / Delete / Set retention).

![Figure 5-15: Key resources to review and clean after the workshop](/images/5-7-cleanup.png)

