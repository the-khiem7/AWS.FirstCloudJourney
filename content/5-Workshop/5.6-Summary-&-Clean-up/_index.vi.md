---
title: "Summary & Clean up"
weight: 56
chapter: false
pre: " <b> 5.6. </b> "
---

## 5.6.1 Tóm tắt

Sau khi hoàn thành bài Lab chúng ta đã dựng được một **Clickstream Analytics Platform** hoàn chỉnh:

1. **Lớp User-Facing**
   - Ứng dụng Next.js (`ClickSteam.NextJS`) trên Amplify + CloudFront  
   - Xác thực người dùng bằng Cognito  
   - PostgreSQL OLTP (`clickstream_web`) trên `SBW_EC2_WebDB` (public subnet)  

2. **Lớp Ingestion & Raw Data**
   - API Gateway HTTP API: `clickstream-http-api` (route `POST /clickstream`)  
   - Lambda Ingest: `clickstream-lambda-ingest`  
   - S3 Raw bucket: `clickstream-s3-ingest/events/YYYY/MM/DD/event-<uuid>.json`  

3. **Lớp Analytics Private**
   - VPC với public & private subnets (`SBW_Project_VPC`)  
   - S3 Gateway Endpoint, SSM Interface Endpoints  
   - Data Warehouse trên EC2: `SBW_EC2_ShinyDWH`, DB `clickstream_dw`  
   - ETL Lambda trong VPC: `SBW_Lamda_ETL`, được trigger bởi `SBW_ETL_HOURLY_RULE`  
   - R Shiny dashboards (`sbw_dashboard`) chỉ truy cập qua SSM port forwarding  

Tổng thể, kiến trúc này cho thấy cách thiết kế một **batch-based analytics platform** an toàn, tối ưu chi phí, chủ yếu dùng serverless + hai EC2.

---

## 5.6.2 Nội dung chính 

- **Separation of concerns**:
  - OLTP và Analytics tách trên 2 EC2 khác nhau, thuộc các domain logic khác nhau.  
- **Security**:
  - DW và Shiny chạy trong private subnet, không có public IP.  
  - SSM Session Manager thay thế SSH truyền thống.  
  - S3 Gateway Endpoint giữ traffic S3 trong private network của AWS.  
- **Tối ưu chi phí**:
  - Không sử dụng NAT Gateway.  
  - ETL dùng serverless (Lambda + EventBridge).  
  - S3 làm storage giá rẻ cho dữ liệu thô.  
- **Dễ mở rộng**:
  - Thiết kế hiện tại là batch-based, nhưng có thể mở rộng sang real-time, analytics phức tạp hơn hoặc chuyển sang các công nghệ DW khác.

---

## 5.6.3 Dọn dẹp Resource

1. **Amplify & CloudFront**
   - Xóa Amplify app (`ClickSteam.NextJS`).  
   - Thao tác này cũng xóa CloudFront distribution.

2. **API Gateway & Lambda**
   - Xóa `clickstream-http-api`.  
   - Xóa các Lambda:
     - `clickstream-lambda-ingest`  
     - `SBW_Lamda_ETL`  

3. **EventBridge**
   - Xóa rule `SBW_ETL_HOURLY_RULE`.  

4. **S3 Buckets**
   - Làm rỗng (empty) rồi xóa:
     - `clickstream-s3-ingest` (RAW clickstream)  
     - `clickstream-s3-sbw` (assets) nếu không dùng cho dự án khác  

5. **EC2 Instances**
   - Stop hoặc terminate:
     - `SBW_EC2_WebDB`  
     - `SBW_EC2_ShinyDWH`  
   - Release Elastic IP (nếu có gán).

6. **VPC & Networking**
   - Xóa VPC endpoints (S3 Gateway, SSM Interface Endpoints).  
   - Xóa route tables, subnets, Internet Gateway.  
   - Cuối cùng, xóa `SBW_Project_VPC` nếu không còn dùng.

