---
title: "Objectives & Scope"
weight: 51
chapter: false
pre: " <b> 5.1. </b> "
---

### Business Context

The target system is an e-commerce website that sells laptops, monitors, and accessories.  
The business wants to:

- Understand **how users interact** with the website:
  - Which pages they visit  
  - Which products they view  
  - Add-to-cart and checkout events  
- Measure **conversion funnels** from product view → add to cart → checkout  
- Identify:
  - Top-performing products  
  - Peak activity periods (time-of-day, day-of-week)  
- **Isolate analytics** from the production database:
  - No heavy reporting queries on OLTP  
  - No public exposure of internal analytics components  

To achieve this, the platform:


---

### Learning Objectives

#### Architectural understanding

- Explain the overall architecture of a **batch-based clickstream analytics platform** using:
  - Amplify, CloudFront, Cognito, OLTP EC2 in the **user-facing domain**  
  - API Gateway, Lambda Ingest, S3 Raw bucket in the **ingestion & data lake domain**  
  - ETL Lambda, PostgreSQL DW on EC2, R Shiny in the **analytics & DW domain**  
- Justify why the OLTP and Analytics layers are:
  - **Logically** separated (different schemas, different workload types)  
  - **Physically** separated (public vs private subnets, different EC2 instances)  

#### Practical skills

- Send clickstream events from the frontend to API Gateway → Lambda Ingest → S3 Raw (`clickstream-s3-ingest`).  
- Configure a **Gateway VPC Endpoint for S3** and update private route tables so private components can reach S3.  
- Configure and test a **ETL Lambda** (`SBW_Lamda_ETL`) that:
  - Reads raw JSON files from `s3://clickstream-s3-ingest/events/YYYY/MM/DD/`  
  - Transforms events into rows for `clickstream_dw.public.clickstream_events`  
- Connect to the DW (`SBW_EC2_ShinyDWH`) and run sample SQL queries:
  - Event counts  
  - Top products  
  - Basic funnels  
- Access **R Shiny dashboards** via SSM port forwarding and interpret:
  - Funnel charts  
  - Product engagement charts  
  - Time-series activity plots  

#### Security and cost-awareness

- Explain why the design **does not use a NAT Gateway**:
  - S3 access is via **Gateway VPC Endpoint**  
  - SSM access is via **Interface VPC Endpoints**  
- Recognize the key security controls:
  - Public vs private subnets  
  - Security groups between `sg_oltp_webDB`, `sg_Lambda_ETL`, `sg_analytics_ShinyDWH`  
  - Minimal IAM permissions for each Lambda  
  - Zero-SSH admin using AWS Systems Manager Session Manager (no bastion host, no open SSH port).

---

### Scope of the Workshop

This workshop focuses on three core capabilities:

1. **Implementing clickstream ingestion**
   - Capturing browser interactions in the Next.js frontend  
   - Sending events as JSON to API Gateway (`clickstream-http-api`)  
   - Storing raw events as time-partitioned files in `clickstream-s3-ingest`  

2. **Building the private analytics layer**
   - Creating VPC, subnets, route tables, and VPC endpoints  
   - Running `SBW_Lamda_ETL` inside the VPC  
   - Connecting Lambda ETL to the private EC2 Data Warehouse (`SBW_EC2_ShinyDWH`)  

3. **Visualizing analytics with Shiny dashboards**
   - Querying `clickstream_dw` from R Shiny  
   - Displaying funnels, product performance, and time-based trends  
   - Accessing Shiny through **SSM Session Manager port forwarding**  

---

### Out-of-Scope Topics

To keep the lab feasible, we explicitly do **not** cover:

- Real-time streaming (Kinesis, Kafka, MSK, etc.)  
- Advanced DW services (Amazon Redshift / Redshift Serverless)  
- ML-based recommendations, segmentation, or anomaly detection  
- Production-grade CI/CD, blue/green deployments, or multi-account setups  
- Heavy SQL tuning and index design  

These are natural future enhancements once the batch-based clickstream foundation is in place.


