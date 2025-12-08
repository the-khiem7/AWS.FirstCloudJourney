---
title: "Workshop"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Nền tảng Phân tích Clickstream cho Thương mại Điện tử

## Tổng quan

Workshop này triển khai **Nền tảng Phân tích Clickstream theo Batch** cho website thương mại điện tử bán sản phẩm máy tính.

Hệ thống thu thập sự kiện clickstream từ frontend, lưu trữ dữ liệu JSON thô trong **Amazon S3**, xử lý sự kiện qua ETL theo lịch (AWS Lambda + EventBridge), và tải dữ liệu phân tích vào **PostgreSQL Data Warehouse** chuyên dụng trên EC2.

Dashboard phân tích được xây dựng bằng **R Shiny**, triển khai trong private subnet và truy vấn trực tiếp Data Warehouse.

Nền tảng được thiết kế với:

- Tách bạch rõ ràng giữa khối lượng công việc **OLTP và Analytics**
- Backend phân tích hoàn toàn riêng tư (không có DW công khai)
- Các thành phần AWS serverless tiết kiệm chi phí, có khả năng mở rộng
- Tối thiểu các thành phần di động để đảm bảo độ tin cậy và đơn giản
- Truy cập quản trị Zero-SSH qua **AWS Systems Manager Session Manager** vào DW riêng tư

---

## Tổng quan Kiến trúc

### 1. User-Facing Domain

**Frontend**: Next.js trên AWS Amplify với CloudFront CDN  
**Xác thực**: Amazon Cognito User Pool  
**OLTP Database**: PostgreSQL trên EC2 (Public Subnet)

### 2. Ingestion & Data Lake Domain

**API Gateway**: HTTP API endpoint cho sự kiện clickstream  
**Lambda Ingest**: Xác thực và ghi JSON thô vào S3  
**S3 Raw Bucket**: Lưu trữ dữ liệu clickstream phân vùng theo thời gian

### 3. Analytics & Data Warehouse Domain

**Lambda ETL**: VPC-enabled, lập lịch bởi EventBridge  
**Data Warehouse**: PostgreSQL trên EC2 (Private Subnet)  
**R Shiny Server**: Dashboard tương tác (Private Subnet)  
**Admin Access**: AWS Systems Manager Session Manager (không SSH)

---

## Tech Stack

### Dịch vụ AWS
- **AWS Amplify Hosting** — Host Next.js (SSR + static assets)
- **Amazon CloudFront** — Phân phối CDN toàn cầu
- **Amazon Cognito** — Xác thực và quản lý danh tính người dùng
- **Amazon S3** — Static assets + dữ liệu clickstream thô
- **Amazon API Gateway (HTTP API)** — Endpoint thu thập sự kiện
- **AWS Lambda** — Serverless compute cho ingestion & ETL
- **Amazon EventBridge** — Lập lịch ETL triggers
- **Amazon EC2** — OLTP DB + Data Warehouse + Shiny
- **Amazon VPC** — Cách ly mạng (public & private subnets)
- **AWS IAM** — Kiểm soát truy cập
- **Amazon CloudWatch** — Logging & monitoring
- **AWS Systems Manager** — Admin tunneling vào private EC2

### Cơ sở dữ liệu
- **PostgreSQL (OLTP)** — Cơ sở dữ liệu vận hành
- **PostgreSQL (DW)** — Data warehouse phân tích

### Analytics
- **R Shiny Server** — Dashboard tương tác
- **Custom ETL** — Lambda chuyển đổi S3 JSON → bảng SQL

---

## Tính năng Chính

- **Batch clickstream ingestion** qua API Gateway + Lambda + S3  
- **Serverless ETL** với EventBridge scheduling  
- **Tách biệt OLTP & Analytics** workloads  
- **Private analytics backend** — hoàn toàn cách ly  
- **Zero-SSH admin access** qua Session Manager  
- **Tối ưu chi phí** — Không NAT Gateway, dùng S3 Gateway Endpoint  
- **Kết nối PostgreSQL trực tiếp** từ Amplify dùng Prisma

---

## Yêu cầu Tiên quyết

Trước khi bắt đầu các phần chi tiết (5.1–5.6), bạn nên có:

- Hiểu biết cơ bản về **dịch vụ AWS** (EC2, S3, Lambda, API Gateway, VPC, IAM)
- Kiến thức thực hành về **SQL** và **PostgreSQL**
- Hiểu biết về **ứng dụng web** (HTTP, JSON, REST APIs)
- Tài khoản AWS với quyền tạo VPC endpoints, Lambda functions, EC2 instances, S3 buckets, và EventBridge rules

> **Lưu ý**: Workshop này giả định rằng hạ tầng cốt lõi (VPC, subnets, EC2, Lambda, API Gateway, S3) đã được cung cấp sẵn qua Infrastructure-as-Code (Terraform/CloudFormation).

---

## Nội dung

1. [Mục tiêu & Phạm vi](5.1-Objectives-&-Scope) — Bối cảnh nghiệp vụ và mục tiêu học tập
2. [Kiến trúc Chi tiết](5.2-Architecture-Walkthrough) — Kiến trúc kỹ thuật chi tiết
3. [Triển khai Thu thập Clickstream](5.3-Implementing-Clickstream-Ingestion) — Thiết lập API Gateway + Lambda
4. [Xây dựng Lớp Phân tích Riêng tư](5.4-Building-the-Private-Analytics-Layer) — ETL Lambda + Data Warehouse
5. [Trực quan hóa bằng Shiny Dashboards](5.5-Visualizing-Analytics-with-Shiny-Dashboards) — Triển khai R Shiny
6. [Tổng kết & Dọn dẹp](5.6-Summary-&-Clean-up) — Bài học chính và dọn dẹp tài nguyên
