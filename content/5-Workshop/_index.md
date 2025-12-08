---
title: "Workshop"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Clickstream Analytics Platform for E-Commerce
 
![Architecture](https://raw.githubusercontent.com/the-khiem7/AWS.FirstCloudJourney/main/static/images/workshop/architecture.png)

#### Overview

This workshop implements a **Batch-Based Clickstream Analytics Platform** for an e-commerce website selling computer products.

The system collects clickstream events from the frontend, stores raw JSON data in **Amazon S3**, processes events via scheduled ETL (AWS Lambda + EventBridge), and loads analytical data into a dedicated **PostgreSQL Data Warehouse** on EC2.

Analytics dashboards are built using **R Shiny**, deployed in a private subnet and directly querying the Data Warehouse.

The platform is engineered with:

- Clear separation between **OLTP vs Analytics** workloads  
- Private-only analytical backend (no public DW access)  
- Cost-efficient, scalable AWS serverless components  
- Minimal moving parts for reliability and simplicity  
- Zero-SSH admin access via **AWS Systems Manager Session Manager** into the private DW

#### Key Architecture Components

**Frontend & OLTP Domain:**
- **Next.js** on AWS Amplify Hosting with CloudFront CDN
- **Amazon Cognito** for user authentication
- **PostgreSQL on EC2** (public subnet) for operational database

**Ingestion & Data Lake Domain:**
- **API Gateway** + **Lambda Ingest** for event collection
- **Amazon S3** raw clickstream bucket (partitioned by date/hour)
- **EventBridge** cron-based batch ETL scheduling

**Analytics & Data Warehouse Domain:**
- **Lambda ETL** (VPC-enabled) processes raw events from S3
- **PostgreSQL Data Warehouse** on EC2 (private subnet)
- **R Shiny Server** for interactive dashboards
- **S3 Gateway VPC Endpoint** for cost-efficient private S3 access (no NAT Gateway)
- **SSM Interface Endpoints** for secure admin access

#### Prerequisites

Before starting the detailed sections (5.1–5.6), you should have:

- Basic familiarity with **AWS services**: EC2, S3, Lambda, API Gateway, VPC, IAM, EventBridge
- Working knowledge of **SQL** and **PostgreSQL**
- Understanding of **web applications** (HTTP, JSON, REST APIs)
- An AWS account with permissions to create VPC endpoints, Lambda functions, EC2 instances, S3 buckets, and EventBridge rules

> **Note**: This workshop assumes core infrastructure (VPC, subnets, EC2 instances, Lambda functions, API Gateway, S3 buckets) is already provisioned via Infrastructure-as-Code (Terraform/CloudFormation).

#### Content Map

1. [Objectives & Scope](5.1-Objectives-&-Scope) - Business context and learning goals
2. [Architecture Walkthrough](5.2-Architecture-Walkthrough) - Detailed component breakdown
3. [Implementing Clickstream Ingestion](5.3-Implementing-Clickstream-Ingestion) - API Gateway + Lambda Ingest
4. [Building the Private Analytics Layer](5.4-Building-the-Private-Analytics-Layer) - ETL Lambda + Data Warehouse
5. [Visualizing Analytics with Shiny Dashboards](5.5-Visualizing-Analytics-with-Shiny-Dashboards) - R Shiny dashboards
6. [Summary & Clean up](5.6-Summary-&-Clean-up) - Key learnings and resource cleanup