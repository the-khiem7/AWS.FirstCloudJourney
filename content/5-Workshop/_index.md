---
title: "Workshop"
menuTitle: "Workshop"
weight: 5
chapter: false
pre: "<b>5. </b>"
---

# Batch-Based Clickstream Analytics Platform

![Architecture](/images/5-Workshop/5-0-architecture.png)
<p align="center"><em>Figure: Architecture Batch-base Clickstream Analytics Platform.</em></p>

#### Overview

This workshop implements a **Batch-Based Clickstream Analytics Platform** for an e-commerce website selling computer products.

The system collects clickstream events from the frontend, stores raw JSON data in **Amazon S3**, processes events via scheduled ETL (**AWS Lambda + EventBridge**), and loads analytical data into a dedicated **PostgreSQL Data Warehouse on EC2** inside a private subnet.

Analytics dashboards are built using **R Shiny**, running on the same EC2 instance as the Data Warehouse, and accessed via **AWS Systems Manager Session Manager**.

The platform is engineered with:

- Clear separation between **OLTP vs Analytics** workloads  
- Private-only analytical backend (**no public DW access**)  
- Cost-efficient, scalable AWS serverless components  
- Zero-SSH admin access via **SSM Session Manager** into the private DW / Shiny EC2 
- You can run the Shiny app locally at **localhost:3838**

#### Key Architecture Components

**Frontend & OLTP Domain**

- Next.js app: **`ClickSteam.NextJS`** hosted on **AWS Amplify Hosting**  
- **Amazon CloudFront** as global CDN  
- **Amazon Cognito** User Pool for authentication  
- OLTP PostgreSQL on EC2: **`SBW_EC2_WebDB`** (public subnet)  
  - DB: `clickstream_web` (schema `public`)  
  - Port: `5432`  

**Ingestion & Data Lake Domain**

- **Amazon API Gateway (HTTP API)**: `clickstream-http-api`  
  - Route: `POST /clickstream`  
- **Lambda Ingest**: `clickstream-lambda-ingest`  
  - Validates payload, enriches metadata, writes JSON files to S3  
- **S3 Raw Clickstream Bucket**: `clickstream-s3-ingest`  
  - Prefix: `events/YYYY/MM/DD/`  
  - File pattern: `event-<uuid>.json`  
  - `RAW_BUCKET = clickstream-s3-ingest`  

**Analytics & Data Warehouse Domain**

- **Private EC2 for DWH + Shiny**: `SBW_EC2_ShinyDWH` (private subnet `10.0.128.0/20`)  
  - DWH DB: `clickstream_dw` 
  - Main table: `clickstream_events` with fields:
    - `event_id, event_timestamp, event_name`  
    - `user_id, user_login_state, identity_source, client_id, session_id, is_first_visit`  
    - `context_product_id, context_product_name, context_product_category, context_product_brand`  
    - `context_product_price, context_product_discount_price, context_product_url_path`  
  - R Shiny Server on port `3838`, web path `/sbw_dashboard`  

- **Lambda ETL**: `SBW_Lamda_ETL` (VPC-enabled)  
  - Reads raw JSON from `clickstream-s3-ingest`  
  - Transforms into SQL-ready rows  
  - Inserts into `clickstream_dw.public.clickstream_events`  

- **EventBridge Rule**: `SBW_ETL_HOURLY_RULE`  
  - Schedule: `rate(1 hour)`  

- **VPC & Networking**

  - VPC CIDR: `10.0.0.0/16`  
  - Public subnet: `10.0.0.0/20` → `SBW_Project-subnet-public1-ap-southeast-1a` (OLTP EC2)  
  - Private subnet: `10.0.128.0/20` → `SBW_Project-subnet-private1-ap-southeast-1a` (DW, Shiny, ETL Lambda)  
  - **S3 Gateway VPC Endpoint** for private S3 access  
  - **SSM Interface Endpoints** (SSM, SSMMessages, EC2Messages) for Session Manager  

- **Admin Access (SSM)**  
  - Port forwarding:
    - `localPort = 3838`  
    - `portNumber = 3838`  
  - Shiny URL from local: `http://localhost:3838/sbw_dashboard`  

#### Content Map

1. **[5.1. Objectives & Scope](5.1-objectives--scope/)**  
2. **[5.2. Architecture Walkthrough](5.2-architecture-walkthrough/)**  
3. **[5.3. Implementing Clickstream Ingestion](5.3-implementing-clickstream-ingestion/)**  
4. **[5.4. Building the Private Analytics Layer](5.4-building-private-analytics-layer/)**  
5. **[5.5. Visualizing Analytics with Shiny Dashboards](5.5-visualizing-analytics-with-shiny-dashboards/)**  
6. **[5.6. Summary & Clean up](5.6-summary-cleanup/)**
