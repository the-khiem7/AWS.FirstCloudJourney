---
title: "Summary & Clean up"
weight: 56
chapter: false
pre: " <b> 5.6. </b> "
---

## 5.6.1 Summary

By completing the workshop and the LAB0–LAB5 exercises, you assembled a full **Clickstream Analytics Platform**:

1. **User-Facing Layer**
   - Next.js app (`ClickSteam.NextJS`) on Amplify + CloudFront  
   - Cognito-based authentication  
   - OLTP PostgreSQL (`clickstream_web`) on `SBW_EC2_WebDB` in a public subnet  

2. **Ingestion & Raw Data Layer**
   - API Gateway HTTP API: `clickstream-http-api` (`POST /clickstream`)  
   - Lambda Ingest: `clickstream-lambda-ingest`  
   - S3 Raw bucket: `clickstream-s3-ingest/events/YYYY/MM/DD/event-<uuid>.json`  

3. **Private Analytics Layer**
   - VPC with public and private subnets (`SBW_Project_VPC`)  
   - S3 Gateway Endpoint, SSM Interface Endpoints  
   - Data Warehouse on EC2: `SBW_EC2_ShinyDWH`, DB `clickstream_dw`  
   - ETL Lambda in VPC: `SBW_Lamda_ETL`, triggered by `SBW_ETL_HOURLY_RULE`  
   - R Shiny dashboards (`sbw_dashboard`) accessible only via SSM port forwarding  

Altogether, this shows how to design a **secure, cost-aware, batch-based analytics platform** using mostly serverless components plus two EC2 instances.

---

## 5.6.2 Main content

- **Separation of concerns**:
  - OLTP vs Analytics on separate EC2 instances and in different logical domains.  
- **Security**:
  - Data Warehouse and Shiny run in a private subnet with no public IP.  
  - SSM Session Manager eliminates the need for SSH.  
  - S3 Gateway Endpoint keeps S3 traffic on AWS private network.  
- **Cost optimization**:
  - No NAT Gateway.  
  - Serverless ETL (Lambda + EventBridge).  
  - S3 as cheap raw data storage.  
- **Extensibility**:
  - Current design is batch-based, but can be extended to real-time streaming, more complex analytics, or different warehouse technologies later.

---

## 5.6.3 Clean Up Resources

1. **Amplify & CloudFront**
   - Remove the Amplify app (`ClickSteam.NextJS`).  
   - This also deletes the associated CloudFront distribution and S3 hosting bucket created by Amplify.

2. **API Gateway & Lambda**
   - Delete `clickstream-http-api`.  
   - Delete Lambda functions:
     - `clickstream-lambda-ingest`  
     - `SBW_Lamda_ETL`  

3. **EventBridge**
   - Delete rule `SBW_ETL_HOURLY_RULE`.  

4. **S3 Buckets**
   - Empty and delete:
     - `clickstream-s3-ingest` (RAW clickstream)  
     - `clickstream-s3-sbw` (assets), if not needed for other projects  

5. **EC2 Instances**
   - Stop or terminate:
     - `SBW_EC2_WebDB`  
     - `SBW_EC2_ShinyDWH`  
   - Release any Elastic IPs associated with them (if used).

6. **VPC & Networking**
   - Delete VPC endpoints (S3 Gateway, SSM Interface Endpoints).  
   - Delete route tables, subnets, Internet Gateway.  
   - Finally, delete `SBW_Project_VPC` if no longer needed.

