---
title: "Architecture Walkthrough"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

This section provides a high-level walkthrough of the overall architecture before diving into the hands-on steps.

### User-facing Domain

- Frontend: **Next.js** hosted on **AWS Amplify**, fronted by **Amazon CloudFront**.  
- Authentication: end users sign in via an **Amazon Cognito User Pool**.  
- Operational database: **PostgreSQL OLTP** running on an **EC2 instance** in the **public subnet**.  
- Connectivity: Amplify connects to the OLTP EC2 instance via the Internet Gateway using Prisma, with `DATABASE_URL` pointing to the public endpoint of the EC2 instance.

> In this design, the OLTP database is intentionally left in a public subnet to keep the connection from Amplify simple. In a production-grade system, OLTP would typically reside in private subnets behind an internal API layer.

### Ingestion & Data Lake Domain

- The frontend sends HTTP events to **Amazon API Gateway (HTTP API)**, route `POST /clickstream`.  
- A **Lambda Ingest** function:
  - Validates the incoming payload.  
  - Enriches metadata such as timestamps, user/session information, and product context.  
  - Writes **raw JSON** into an **S3 Raw Clickstream bucket** using a partitioned directory structure:

  ```text
  s3://<raw-bucket>/events/YYYY/MM/DD/HH/events-<uuid>.json
  ```

This pattern keeps raw data immutable and time-partitioned, which is convenient for batch ETL jobs.

### Analytics & Data Warehouse Domain

- A **VPC-enabled ETL Lambda** runs inside a **private subnet** dedicated to analytics/ETL.  
- The ETL Lambda reads data from S3 via an **S3 Gateway VPC Endpoint**, then:
  - Cleans and normalizes events.  
  - Converts JSON logs into SQL-friendly analytical tables (facts and dimensions).  
  - Writes data into a **PostgreSQL Data Warehouse** hosted on an EC2 instance in a **separate private subnet**.

- An **R Shiny Server** runs on the same EC2 instance as the Data Warehouse and connects through localhost/private IP. It renders interactive dashboards on top of the DW schema.
  
**Figure 5-2: EC2 instances and basic CloudWatch metrics**

The screenshot shows the two EC2 instances used in the workshop:

- `SBW_EC2_Web_OLTP` – the public instance that hosts the OLTP PostgreSQL database.  
- `SBW_EC2_Shiny_DW` – the private instance that hosts the PostgreSQL Data Warehouse and the Shiny Server.

Below the instance list, the EC2 console displays basic CloudWatch metrics such as CPU utilization and network traffic, which can be used to verify that the instances are healthy while the workshop labs are running.

![Figure 5-2: EC2 instances and basic CloudWatch metrics](/images/5-b-ec2-instances-metrics.png)

**Figure 5-3: Overall Clickstream Analytics Architecture**  

The diagram illustrates:

- Outside the VPC: Amazon Cognito, Amazon CloudFront, AWS Amplify, Amazon API Gateway, Lambda Ingest, and Amazon EventBridge.  
- Inside the VPC:  
  - **Public Subnet – OLTP**: EC2 PostgreSQL OLTP instance and the Internet Gateway.  
  - **Private Subnet – Analytics**: EC2 PostgreSQL Data Warehouse and R Shiny Server (no public IP).  
  - **Private Subnet – ETL**: VPC-enabled ETL Lambda function and the S3 Gateway VPC Endpoint.  
- Numbered arrows (1)–(13) show the main flows: user login, browsing, clickstream ingestion, batch ETL, loading data into the DW, and Shiny visualization.

![Figure 5-3: Overall Clickstream Analytics Architecture for the e-commerce platform](/images/5-1-clickstream-architecture.png)


**Figure 5-4: VPC Network Layout (OLTP & Analytics)**

The diagram illustrates a single VPC with two subnets:

- **Public Subnet – OLTP (10.0.1.0/24)**, highlighted in yellow, hosting the EC2 PostgreSQL OLTP instance and connected to the Internet Gateway.  
- **Private Subnet – Analytics (10.0.2.0/24)**, in green, hosting the EC2 PostgreSQL Data Warehouse, the R Shiny Server, the VPC-enabled ETL Lambda function, and the S3 Gateway VPC Endpoint (all with no public IP).

The public subnet route table includes a default route:

- `0.0.0.0/0 → Internet Gateway (IGW)`

The private subnet route table:

- Has `10.0.0.0/16 → local` for internal VPC traffic.  
- Includes a route to the S3 prefix list via the **S3 Gateway VPC Endpoint**.  
- Does **not** have any `0.0.0.0/0` route and does **not** use a NAT Gateway.

![Figure 5-4: VPC network layout for OLTP and Analytics](/images/5-2-vpc-network.png)
