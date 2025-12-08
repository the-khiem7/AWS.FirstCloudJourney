---
title: "Architecture Walkthrough"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Phần này cung cấp một cái nhìn tổng quan ở mức cao về kiến trúc tổng thể trước khi đi vào các bước thực hành chi tiết.

### Miền nghiệp vụ hướng người dùng (User-facing Domain)

- Frontend: **Next.js** được host trên **AWS Amplify**, phía trước là **Amazon CloudFront**.  
- Xác thực: người dùng cuối đăng nhập thông qua **Amazon Cognito User Pool**.  
- Cơ sở dữ liệu vận hành: **PostgreSQL OLTP** chạy trên một **EC2 instance** trong **public subnet**.  
- Kết nối: Amplify kết nối tới EC2 OLTP thông qua Internet Gateway bằng Prisma, với biến môi trường `DATABASE_URL` trỏ đến endpoint public của EC2 instance.

> Trong thiết kế này, cơ sở dữ liệu OLTP được đặt trong public subnet một cách có chủ đích để giữ cho kết nối từ Amplify đơn giản. Trong một hệ thống đạt chuẩn production, OLTP thường sẽ nằm trong các private subnet phía sau một lớp API nội bộ.

### Miền nghiệp vụ hướng ingest & data lake (Ingestion & Data Lake Domain)

- Frontend gửi các sự kiện HTTP tới **Amazon API Gateway (HTTP API)**, route `POST /clickstream`.  
- Một **Lambda Ingest**:
  - Kiểm tra tính hợp lệ (validate) của payload nhận được.  
  - Bổ sung (enrich) các metadata như timestamp, thông tin user/session, và ngữ cảnh sản phẩm.  
  - Ghi **JSON thô** vào **S3 Raw Clickstream bucket** theo cấu trúc thư mục được phân vùng theo thời gian:

  ```text
  s3://<raw-bucket>/events/YYYY/MM/DD/HH/events-<uuid>.json
  ```

Mẫu (pattern) này giữ cho dữ liệu thô bất biến và được phân vùng theo thời gian, rất thuận tiện cho các job ETL theo lô (batch ETL).

### Miền nghiệp vụ hướng phân tích & data warehouse (Analytics & Data Warehouse Domain)

- Một **ETL Lambda chạy trong VPC** được đặt trong **private subnet** dành riêng cho analytics/ETL.  
- ETL Lambda đọc dữ liệu từ S3 thông qua **S3 Gateway VPC Endpoint**, sau đó:
  - Làm sạch (clean) và chuẩn hóa (normalize) các event.  
  - Chuyển đổi log JSON thành các bảng phân tích thân thiện với SQL (bảng fact và dimension).  
  - Ghi dữ liệu vào **PostgreSQL Data Warehouse** host trên một EC2 instance trong một **private subnet riêng biệt**.

- Một **R Shiny Server** chạy trên cùng EC2 instance với Data Warehouse và kết nối qua localhost / private IP. Shiny render các dashboard tương tác dựa trên schema của Data Warehouse.

**Hình 5-2: Các EC2 instance và các metric CloudWatch cơ bản**

Ảnh chụp màn hình cho thấy hai EC2 instance được sử dụng trong workshop:

- `SBW_EC2_Web_OLTP` – instance public host cơ sở dữ liệu PostgreSQL OLTP.  
- `SBW_EC2_Shiny_DW` – instance private host PostgreSQL Data Warehouse và Shiny Server.

Bên dưới danh sách instance, console EC2 hiển thị các CloudWatch metric cơ bản như CPU utilization và network traffic, có thể dùng để xác minh tình trạng “khỏe mạnh” của các instance trong lúc chạy các lab của workshop.

![Hình 5-2: Các EC2 instance và các metric CloudWatch cơ bản](/images/5-b-ec2-instances-metrics.png)

**Hình 5-3: Kiến trúc tổng thể nền tảng Clickstream Analytics**  

Sơ đồ minh họa:

- Bên ngoài VPC: Amazon Cognito, Amazon CloudFront, AWS Amplify, Amazon API Gateway, Lambda Ingest và Amazon EventBridge.  
- Bên trong VPC:  
  - **Public Subnet – OLTP**: EC2 PostgreSQL OLTP và Internet Gateway.  
  - **Private Subnet – Analytics**: EC2 PostgreSQL Data Warehouse và R Shiny Server (không có public IP).  
  - **Private Subnet – ETL**: ETL Lambda chạy trong VPC và S3 Gateway VPC Endpoint.  
- Các mũi tên được đánh số (1)–(13) thể hiện các luồng chính: đăng nhập người dùng, duyệt web, ingest clickstream, ETL theo lô, nạp dữ liệu vào Data Warehouse và trực quan hóa bằng Shiny.

![Hình 5-3: Kiến trúc tổng thể nền tảng Clickstream Analytics cho hệ thống thương mại điện tử](/images/5-1-clickstream-architecture.png)


**Hình 5-4: Sơ đồ mạng VPC (OLTP & Analytics)**

Sơ đồ minh họa một VPC duy nhất với hai subnet:

- **Public Subnet – OLTP (10.0.1.0/24)**, tô màu vàng, host EC2 PostgreSQL OLTP và kết nối tới Internet Gateway.  
- **Private Subnet – Analytics (10.0.2.0/24)**, tô màu xanh lá, host EC2 PostgreSQL Data Warehouse, R Shiny Server, ETL Lambda chạy trong VPC, và S3 Gateway VPC Endpoint (tất cả đều không có public IP).

Route table của public subnet bao gồm route mặc định:

- `0.0.0.0/0 → Internet Gateway (IGW)`

Route table của private subnet:

- Có route `10.0.0.0/16 → local` cho traffic nội bộ trong VPC.  
- Bao gồm một route tới S3 prefix list thông qua **S3 Gateway VPC Endpoint**.  
- **Không** có bất kỳ route `0.0.0.0/0` nào và **không** sử dụng NAT Gateway.

![Hình 5-4: Sơ đồ mạng VPC cho OLTP và Analytics](/images/5-2-vpc-network.png)

---

### Thiết kế Mạng & Bảo mật

#### Cấu hình VPC

- **VPC CIDR**: `10.0.0.0/16`
- **Internet Gateway (IGW)**: Gắn vào VPC, cung cấp kết nối hai chiều cho các tài nguyên trong public subnet

**Các Subnet:**

1. **Public Subnet (10.0.1.0/24) - Lớp OLTP**
   - EC2 PostgreSQL OLTP (có public IP)
   - Định tuyến traffic Internet qua Internet Gateway
   - Cho phép inbound từ Amplify và các IP admin
   - Cho phép outbound để cập nhật và gọi các API bên ngoài

2. **Private Subnet 1 (10.0.2.0/24) - Lớp Analytics**
   - EC2 Data Warehouse (PostgreSQL) - không có public IP
   - EC2 R Shiny Server - không có public IP
   - SSM Interface Endpoint cho Session Manager
   - Không có truy cập internet trực tiếp (không có route tới IGW)
   - Hoàn toàn cô lập khỏi internet công cộng

3. **Private Subnet 2 (10.0.3.0/24) - Lớp ETL**
   - Lambda ETL (chạy trong VPC) - không có public IP
   - S3 Gateway VPC Endpoint để truy cập S3 riêng tư
   - Không có truy cập internet trực tiếp (không có route tới IGW)

#### Bảng định tuyến

**Public Route Table** (Public Subnet):
- `10.0.0.0/16` → Local (nội bộ VPC)
- `0.0.0.0/0` → Internet Gateway (default route)

**Private Route Table 1** (Analytics Subnet):
- `10.0.0.0/16` → Local (chỉ nội bộ VPC)
- **Không có default route tới Internet Gateway**
- Truy cập admin qua SSM Interface Endpoints

**Private Route Table 2** (ETL Subnet):
- `10.0.0.0/16` → Local (nội bộ VPC)
- S3 prefix list → S3 Gateway VPC Endpoint
- **Không có default route tới Internet Gateway**

> **Thiết kế Chính**: Không triển khai NAT Gateway. Các thành phần private truy cập S3 qua Gateway VPC Endpoint, loại bỏ chi phí NAT trong khi vẫn duy trì bảo mật.

#### Security Groups

**SG-OLTP:**
- Inbound: `5432/tcp` từ Amplify/trusted IPs, `22/tcp` từ admin IP
- Outbound: mặc định (tất cả được phép)

**SG-DW:**
- Inbound: `5432/tcp` từ Lambda ETL SG và Shiny SG
- Outbound: `443/tcp` tới SSM interface endpoints
- Truy cập admin qua Session Manager (không có inbound SSH)

**SG-Shiny:**
- Inbound: hạn chế chỉ cho admin/VPN
- Outbound: được phép tới DW (localhost/private IP)

**SG-ETL-Lambda:**
- Không có inbound (Lambda không chấp nhận inbound)
- Outbound: được phép tới S3 endpoint + DW SG

#### Các Dịch vụ AWS Bên ngoài

Các dịch vụ bên ngoài VPC tương tác với hạ tầng:

- **AWS Amplify Hosting**: Kết nối tới OLTP EC2 qua Internet Gateway bằng Prisma
- **Amazon CloudFront**: Phân phối static assets trên toàn cầu
- **Amazon Cognito**: Dịch vụ xác thực khu vực, không tương tác với VPC
- **Amazon API Gateway**: Điểm vào cho clickstream events, gọi Lambda Ingest
- **Amazon EventBridge**: Kích hoạt các ETL jobs theo lịch (Lambda ETL trong VPC)
- **AWS Systems Manager**: Truy cập admin qua VPC Interface Endpoints (không SSH)

---

### Tóm tắt Luồng Dữ liệu

**Luồng Tương tác Người dùng:**
1. Người dùng truy cập qua CloudFront → Amplify Hosting
2. Người dùng xác thực qua Cognito
3. Amplify SSR/API truy vấn OLTP EC2 qua Internet Gateway bằng Prisma

**Ingestion Clickstream:**
4. Frontend gửi events tới API Gateway
5. Lambda Ingest ghi raw JSON vào S3 Raw Clickstream Bucket

**Xử lý ETL theo Lô:**
6. EventBridge (lịch cron, ví dụ: mỗi 30 phút) kích hoạt Lambda ETL
7. Lambda ETL (chạy trong VPC tại Private Subnet 2):
   - Đọc raw files từ S3 qua Gateway VPC Endpoint
   - Làm sạch, chuẩn hóa, sessionize các events
   - Kết nối tới PostgreSQL DW (Private Subnet 1) qua VPC routing
   - Chèn dữ liệu đã xử lý vào các bảng DW

**Truy cập Analytics:**
8. R Shiny Server (cùng EC2 với DW) kết nối qua localhost
9. Admin sử dụng AWS Systems Manager Session Manager port-forwarding qua SSM interface endpoints

---

### Các Tính năng Chính

- Ingestion clickstream theo lô (API Gateway + Lambda + S3)  
- ETL serverless với lịch EventBridge  
- Tách biệt rõ ràng giữa OLTP và Analytics  
- R Shiny dashboards (hoàn toàn private)  
- Zero-SSH admin qua SSM Session Manager  
- Tối ưu chi phí (không NAT Gateway, S3 Gateway Endpoint, Lambda compute)  
- Kết nối PostgreSQL trực tiếp từ Amplify bằng Prisma

---

### Tóm tắt Tech Stack

**Các Dịch vụ AWS:**
- AWS Amplify Hosting, CloudFront, Cognito
- Amazon S3, API Gateway, Lambda, EventBridge
- Amazon EC2, VPC, IAM, CloudWatch
- AWS Systems Manager (Session Manager + VPC Endpoints)

**Cơ sở dữ liệu:**
- PostgreSQL (EC2 OLTP) - cơ sở dữ liệu vận hành
- PostgreSQL (EC2 DW) - cơ sở dữ liệu phân tích

**Analytics:**
- R Shiny Server - dashboard tương tác
- Logic ETL tùy chỉnh - Lambda chuyển đổi S3 JSON → bảng SQL
