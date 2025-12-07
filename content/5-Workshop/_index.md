---
title: "Workshop"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Clickstream Analytics Platform for E-Commerce

#### Overview

This workshop walks through a **Batch-based Clickstream Analytics Platform** for an e-commerce website that sells computer products.

You will see how clickstream events:

- Are captured by a **Next.js** frontend hosted on **AWS Amplify** and delivered through **Amazon CloudFront**.  
- Are ingested via **Amazon API Gateway** and a **Lambda Ingest** function into a **Raw Clickstream bucket on Amazon S3**.  
- Are processed in batch by a **VPC-enabled ETL Lambda** that reads from S3 through an **S3 Gateway VPC Endpoint** and loads curated data into a **PostgreSQL Data Warehouse** on **EC2** in a private subnet.  
- Are visualized via **R Shiny dashboards** running on the same EC2 instance as the Data Warehouse.

The workshop emphasises:

- **Separation of OLTP and Analytics** workloads.  
- A fully **private analytics backend** (ETL Lambda, Data Warehouse, Shiny) with no public IPs.  
- **Cost‑efficient private access to S3** using a Gateway VPC Endpoint instead of a NAT Gateway.

#### Prerequisites

Before starting the detailed sections (5.1–5.6), the reader is expected to have:

- Basic familiarity with **AWS services** such as EC2, S3, Lambda, API Gateway, VPC, and IAM.  
- Working knowledge of **SQL** and **PostgreSQL**.  
- A general understanding of **web applications** (HTTP, JSON, REST APIs).  
- An AWS account with sufficient permissions to create VPC endpoints, Lambda functions, EC2 instances, S3 buckets, and EventBridge rules.

> This workshop assumes that the core infrastructure (VPC, subnets, EC2 instances, Lambda functions, API Gateway, and S3 buckets) is already provisioned, for example via Infrastructure‑as‑Code such as Terraform or CloudFormation.

#### Content Map

The workshop content is split into six sections:

1. [Objectives & Scope](5.1-Objectives-&-Scope)
2. [Architecture Walkthrough](5.2-Architecture-Walkthrough) 
3. [Implementing Clickstream Ingestion](5.3-Implementing-Clickstream-Ingestion) 
4. [Building the Private Analytics Layer](5.4-Building-the-Private-Analytics-Layer)  
5. [Visualizing Analytics with Shiny Dashboards](5.5-Visualizing-Analytics-with-Shiny-Dashboards)
6. [Summary & Clean up](5.6-Summary-&-Clean-up)