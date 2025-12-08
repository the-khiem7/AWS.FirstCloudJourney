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

- `SBW_EC2_WebDB` – the public instance that hosts the OLTP PostgreSQL database.  
- `SBW_EC2_ShinyDWH` – the private instance that hosts the PostgreSQL Data Warehouse and the Shiny Server.

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

- **Public Subnet – OLTP (10.0.0.0/20)**, highlighted in yellow, hosting the EC2 PostgreSQL OLTP instance and connected to the Internet Gateway.  
- **Private Subnet – Analytics & ETL (10.0.128.0/20)**, in green, hosting the EC2 PostgreSQL Data Warehouse, the R Shiny Server, the VPC-enabled ETL Lambda function, and the S3 Gateway VPC Endpoint (all with no public IP).

The public subnet route table includes a default route:

- `0.0.0.0/0 → Internet Gateway (IGW)`

The private subnet route table:

- Has `10.0.0.0/16 → local` for internal VPC traffic.  
- Includes a route to the S3 prefix list via the **S3 Gateway VPC Endpoint**.  
- Does **not** have any `0.0.0.0/0` route and does **not** use a NAT Gateway.

![Figure 5-4: VPC network layout for OLTP and Analytics](/images/5-2-vpc-network.png)

---

### Networking & Security Design

#### VPC Layout

- **VPC CIDR**: `10.0.0.0/16`
- **Internet Gateway (IGW)**: Attached to VPC, provides bidirectional connectivity for public subnet resources

**Subnets:**

1. **Public Subnet (10.0.0.0/20) - OLTP Layer**
   - EC2 PostgreSQL OLTP (with public IP)
   - Routes internet traffic via Internet Gateway
   - Allows inbound from Amplify and admin IPs
   - Allows outbound for updates and external APIs

2. **Private Subnet (10.0.128.0/20) - Analytics & ETL Layer**
   - EC2 Data Warehouse (PostgreSQL) - no public IP
   - EC2 R Shiny Server - no public IP
   - SSM Interface Endpoint for Session Manager
   - No direct internet access (no route to IGW)
   - Fully isolated from public internet



#### Routing Tables

**Public Route Table** (Public Subnet):
- `10.0.0.0/16` → Local (VPC internal)
- `0.0.0.0/0` → Internet Gateway (default route)

**Private Route Table** (Analytics & ETL Subnet):
- `10.0.0.0/16` → Local (VPC internal only)
- S3 prefix list → S3 Gateway VPC Endpoint
- **No default route to Internet Gateway**
- Admin access via SSM Interface Endpoints

> **Key Design**: No NAT Gateway deployed. Private components reach S3 via Gateway VPC Endpoint, eliminating NAT costs while maintaining security.

#### Security Groups

**sg_oltp_webDB:**
- Inbound: `5432/tcp` from Amplify/trusted IPs, `22/tcp` from admin IP
- Outbound: default (all allowed)

**sg_analytics_ShinyDWH:**
- Inbound: `5432/tcp` from Lambda ETL SG and Shiny SG
- Outbound: `443/tcp` to SSM interface endpoints
- Admin access via Session Manager (no inbound SSH)

**sg_Lambda_ETL:**
- No inbound (Lambda doesn't accept inbound)
- Outbound: allowed to S3 endpoint + DWH SG

#### External AWS Services

Services outside VPC that interact with infrastructure:

- **AWS Amplify Hosting**: Connects to OLTP EC2 via Internet Gateway using Prisma
- **Amazon CloudFront**: Distributes static assets globally
- **Amazon Cognito**: Regional auth service, no VPC interaction
- **Amazon API Gateway**: Entry point for clickstream events, invokes Lambda Ingest
- **Amazon EventBridge**: Triggers scheduled ETL jobs (Lambda ETL in VPC)
- **AWS Systems Manager**: Admin access via VPC Interface Endpoints (no SSH)

---

### Data Flow Summary

**User Interaction Flow:**
1. User accesses via CloudFront → Amplify Hosting
2. User authenticates via Cognito
3. Amplify SSR/API queries OLTP EC2 via Internet Gateway using Prisma

**Clickstream Ingestion:**
4. Frontend sends events to API Gateway
5. Lambda Ingest writes raw JSON to S3 Raw Clickstream Bucket

**Batch ETL Processing:**
6. EventBridge (cron schedule, e.g. every 30 min) triggers Lambda ETL
7. Lambda ETL (VPC-enabled in Private Subnet 2):
   - Reads raw files from S3 via Gateway VPC Endpoint
   - Cleans, normalizes, sessionizes events
   - Connects to PostgreSQL DW (Private Subnet 1) via VPC routing
   - Inserts processed data into DW tables

**Analytics Access:**
8. R Shiny Server (same EC2 as DW) connects via localhost
9. Admin uses AWS Systems Manager Session Manager port-forwarding via SSM interface endpoints

---

### Key Features

- Batch clickstream ingestion (API Gateway + Lambda + S3)  
- Serverless ETL with EventBridge scheduling  
- Clear OLTP vs Analytics separation  
- R Shiny dashboards (fully private)  
- Zero-SSH admin via SSM Session Manager  
- Cost-optimized (no NAT Gateway, S3 Gateway Endpoint, Lambda compute)  
- Direct PostgreSQL connectivity from Amplify using Prisma

---

### Tech Stack Summary

**AWS Services:**
- AWS Amplify Hosting, CloudFront, Cognito
- Amazon S3, API Gateway, Lambda, EventBridge
- Amazon EC2, VPC, IAM, CloudWatch
- AWS Systems Manager (Session Manager + VPC Endpoints)

**Databases:**
- PostgreSQL (EC2 OLTP) - `clickstream_web` operational database
- PostgreSQL (EC2 DWH) - `clickstream_dw` analytical database

**Analytics:**
- R Shiny Server - interactive dashboards
- Custom ETL logic - Lambda transforming S3 JSON → SQL tables
