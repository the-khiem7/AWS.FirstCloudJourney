---
title: "Objectives & Scope"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

This section defines the **purpose**, **learning objectives**, and **scope** of the workshop that is built around a Batch-based Clickstream Analytics Platform for an e-commerce website selling computer products.

The platform is implemented on AWS and uses:

- Next.js on **AWS Amplify Hosting** (front-end, Server-Side Rendering).  
- **Amazon CloudFront** (global content delivery).  
- **Amazon Cognito** (user authentication).  
- **Amazon API Gateway** and **AWS Lambda** (clickstream ingestion and ETL).  
- **Amazon S3** (raw clickstream data storage).  
- **Amazon VPC** with public and private subnets (network isolation).  
- **PostgreSQL** on **Amazon EC2** (OLTP and Data Warehouse).  
- **R Shiny Server** (analytics dashboards).  
- **AWS Systems Manager** (Session Manager for secure, zero-SSH admin access to private EC2 instances).

---

### Business Context

The target system is an e-commerce website that sells computer-related products (laptops, monitors, accessories, etc.).  

The business would like to:

- Understand **how users interact with the website** (pages visited, products viewed, add-to-cart events, checkout attempts).  
- Measure **conversion funnels** (from product view to purchase).  
- Identify **top-performing products** and periods of high activity.  
- Do this **without impacting** the operational database and **without exposing** internal analytics components to the public Internet.

The platform therefore separates **online transaction processing (OLTP)** from **analytics**, and gathers clickstream data in a dedicated analytics environment.

---

### Learning Objectives

After completing all sections (5.1–5.6), the reader should be able to:

#### Architectural understanding

- Describe the overall architecture of a **batch-based clickstream analytics platform** on AWS.  
- Explain the differences between:
  - The **user-facing domain** (Amplify, CloudFront, Cognito, OLTP EC2).  
  - The **ingestion & data lake domain** (API Gateway, Lambda Ingest, S3 Raw bucket).  
  - The **analytics & data warehouse domain** (ETL Lambda, PostgreSQL DW, Shiny).  
- Justify why OLTP and Analytics are **logically and physically separated**, and how this reduces risk for the operational workload.

#### Practical skills

- Trigger and inspect clickstream ingestion from the frontend to **API Gateway → Lambda Ingest → S3**.  
- Configure a **Gateway VPC Endpoint for S3** and update route tables so that analytics components inside private subnets can reach S3 **without using a NAT Gateway**.  
- Configure and test a **VPC-enabled ETL Lambda** that:
  - Reads raw JSON files from the S3 Raw Clickstream bucket.  
  - Transforms events into SQL-ready analytical tables.  
  - Loads data into a PostgreSQL Data Warehouse hosted on EC2 in a private subnet.  
- Connect to the Data Warehouse and run sample **SQL queries** to validate the pipeline (event counts, top products, etc.).  
- Access **R Shiny dashboards** running on the same EC2 instance as the Data Warehouse, and interpret the main charts (funnels, top products, time series).

#### Security and cost-awareness

- Explain how **Gateway VPC Endpoints** keep S3 traffic on the **AWS private network**.  
- Compare using a Gateway VPC Endpoint versus a **NAT Gateway** in terms of **security**, **cost**, and **operational complexity**.  
- List the main security controls used in the architecture:
  - Public vs. private subnets.  
  - Security groups between OLTP, ETL Lambda, and Data Warehouse.  
  - Limited IAM permissions for Lambda Ingest and ETL Lambda.  
  - **Zero-SSH admin access** using **AWS Systems Manager Session Manager** via VPC Interface Endpoints, eliminating the need for bastion hosts or exposed SSH ports.

---

### Scope of the Workshop

The workshop focuses on **three core capabilities** of the platform:

1. **Implementing clickstream ingestion**  
   - Capturing user interactions in the browser.  
   - Sending events as JSON to API Gateway.  
   - Persisting raw events as time-partitioned files in an S3 Raw Clickstream bucket.

2. **Building the private analytics layer**  
   - Configuring the VPC, private subnets, and S3 Gateway VPC Endpoint.  
   - Running ETL Lambda inside the VPC so that it can:
     - Read from S3 using private connectivity.  
     - Write data into the PostgreSQL Data Warehouse on a private EC2 instance.

3. **Visualizing analytics with Shiny dashboards**  
   - Querying analytical tables from R Shiny Server running on the same EC2 instance as the Data Warehouse.  
   - Providing interactive dashboards for funnel analysis, product performance, and time-based trends.

The workshop assumes that:

- The base infrastructure (VPC, subnets, EC2 instances, IAM roles, basic Lambda skeletons, and S3 buckets) has already been provisioned, for example via **Terraform** or **CloudFormation**.  
- The focus is on **understanding and validating** the data flow and private connectivity, rather than on writing production-grade ETL code or frontend code from scratch.

---

### Out-of-Scope Topics

To keep the content focused and achievable within a limited time, the workshop **does not** cover:

- Real-time streaming pipelines (for example, Amazon Kinesis, Kafka, or managed streaming services).  
- Advanced data warehousing technologies such as **Amazon Redshift**, **Redshift Serverless**, or **Lakehouse** architectures.  
- Complex machine learning or user segmentation models built on top of the clickstream data.  
- Production-grade CI/CD pipelines, blue/green deployments, or multi-account AWS setups.  
- Deep optimization of SQL queries and index design beyond simple examples.

These topics are considered **future extensions** of the platform and may be explored in subsequent work.

---

### Target Audience

The workshop is primarily intended for:

- Students or engineers with a basic understanding of AWS who want to see how multiple services fit together into a real analytics solution.  
- Developers who are comfortable with **JavaScript/TypeScript**, **SQL**, and **Linux-based environments**.  
- Anyone interested in learning how to design a **secure, cost-conscious analytics architecture** using mostly serverless components and a small EC2 footprint.

Readers are not expected to be experts in networking or data engineering; the workshop provides concrete, guided steps in later sections (5.2–5.5) to make the architecture tangible and reproducible.

**Figure 5-1: Amplify hosting for the e-commerce frontend**

The screenshot shows the Amplify application that hosts the Next.js e-commerce frontend, including the main branch, last deployment status, and the CloudFront URL used in this workshop.

![Figure 5-1: Amplify hosting for the e-commerce frontend](/images/5-a-amplify-overview.png)
