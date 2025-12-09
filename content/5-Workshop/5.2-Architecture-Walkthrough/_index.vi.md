---
title: "Architecture Walkthrough"
weight: 52
chapter: false
pre: " <b> 5.2. </b> "
---

![Architecture](/images/5-Workshop/5-0-architecture.png)
<p align="center"><em>Hình: Batch-based Clickstream Analytics Platform.</em></p>

---

## 5.2.1 Miền User & Frontend

### (1) User Browser – Người dùng

**Nhiệm vụ**

- Truy cập website: duyệt catalogue, xem trang chi tiết sản phẩm, giỏ hàng, checkout…
- Đăng nhập / đăng ký tài khoản.
- Phát sinh **clickstream events** như:
  - `page_view`, `product_view`, `add_to_cart`, `purchase`, …

**Vai trò trong pipeline**

- Là **nguồn gốc mọi hành vi** mà hệ thống analytics phân tích.
- JavaScript ở frontend sẽ gom các event này và gửi về backend thông qua API (6).

---

### (2) Amazon CloudFront (CDN / Edge)

**Nhiệm vụ**

- Là lớp **CDN** phân phối nội dung cho website:
  - Cache HTML/CSS/JS/images để giảm latency.
  - Giảm tải cho origin (Amplify, S3 assets).
- Có thể làm **điểm vào chung** cho traffic web:
  - Route/forward request:
    - Nội dung web động → Amplify.
    - Ảnh / media tĩnh → S3 (3).

**Tại sao quan trọng**

- Website thương mại điện tử cần **tốc độ tải trang** tốt.  
- CloudFront giúp cải thiện trải nghiệm người dùng ở nhiều khu vực địa lý.

---

### (3) Amazon S3 – Media Assets

**Nhiệm vụ**

- Lưu **ảnh sản phẩm** và các **assets tĩnh** khác của website.
- Đóng vai trò **origin** khi CloudFront (2) phục vụ ảnh/asset.

**Lưu ý**

- Bucket này (ví dụ `clickstream-s3-sbw`) **tách biệt** khỏi RAW clickstream bucket (8) để:
  - Phân quyền rõ ràng.
  - Dễ quản trị, dễ dọn dẹp.

---

### (4) Amazon Amplify – Front-end Hosting

**Nhiệm vụ**

- Build & deploy ứng dụng **Next.js** (ví dụ `ClickSteam.NextJS`).
- Host frontend (SSR/ISR, API routes).
- Quản lý pipeline deploy: build, artifact, domain mapping.

**Liên kết với dịch vụ khác**

- Kết nối với **Cognito (5)** để xử lý đăng nhập.
- Gửi **clickstream events** từ frontend tới **API Gateway (6)**.
- Kết nối tới **EC2 OLTP (20)** qua Prisma để đọc/ghi dữ liệu giao dịch.

---

### (5) Amazon Cognito – Authentication / Authorization

**Nhiệm vụ**

- Quản lý **user identity**:
  - Đăng ký / đăng nhập / reset password.
  - Lưu user pool, cấp token JWT (ID token, access token).
- Cấp token cho frontend để gọi các API backend theo đúng quyền.

**Vai trò trong clickstream**

- Cho phép xác định được trạng thái:
  - `user_login_state` (logged-in / guest).
  - `identity_source` (Cognito, social login…).
- Các thông tin này được gắn vào event và lưu xuống S3/DWH để phân tích hành vi user.

---

## 5.2.2 Miền Ingestion & Data Lake

### (6) Amazon API Gateway – Clickstream HTTP API

**Nhiệm vụ**

- Cung cấp endpoint HTTP public cho frontend:
  - `POST /clickstream`.
- Thực hiện validate cơ bản:
  - Method/path, throttling.
  - Tùy chọn: tích hợp Cognito authorizer.

**Luồng xử lý**

- Nhận request JSON từ browser.
- Forward payload tới **Lambda Clickstream Ingest (7)**.

---

### (7) AWS Lambda – Clickstream Ingest

**Nhiệm vụ chính**

- Nhận event từ API Gateway (6).
- Đóng vai trò **ingestion layer**:

  - Validate schema / loại event.
  - Enrich tối thiểu:
    - Sinh `event_id`.
    - Chuẩn hoá `event_timestamp`.
    - Gắn `client_id`, `session_id`, `user_login_state`, `identity_source` nếu cần.

- Ghi event thô (`raw JSON`) xuống **S3 Clickstream Raw bucket (8)** theo prefix/time-partition.

- Ghi log ra **CloudWatch (17)** để debug.

---

### (8) Amazon S3 – Clickstream Raw Data

**Nhiệm vụ**

- Lưu toàn bộ dữ liệu **clickstream thô** do Lambda Ingest (7) đổ vào.
- Đóng vai trò một **“mini data lake”** cho batch ETL:

  - Dữ liệu tổ chức theo **folder/prefix thời gian**:
    - `events/YYYY/MM/DD/`.
  - ETL (11) đọc dữ liệu theo lô (theo giờ / theo ngày…).

**Vai trò kiến trúc**

- Là **source of truth** cho clickstream:
  - Có thể reprocess hoặc rebuild Data Warehouse nếu cần.

---

### (9) Amazon EventBridge – ETL Trigger / Cron Job

**Nhiệm vụ**

- Chạy lịch batch schedule, ví dụ: `rate(1 hour)`.
- Mỗi lần đến lịch:
  - Trigger **Lambda ETL (11)**.

**Tại sao tách riêng**

- “Đồng hồ” ETL nằm ở EventBridge:
  - Dễ chỉnh tần suất (1h, 30’, 5’…).
  - Dễ disable/enable.
  - Tách “khi nào chạy” khỏi logic ETL.

---

## 5.2.3 Miền VPC & Private Analytics

### (19) VPC + Subnets + Internet Gateway

**Amazon VPC**

- Cô lập mạng, kiểm soát routing / security cho toàn platform.

**Public subnet – OLTP**

- Chứa **EC2 OLTP (20)**.
- Có route `0.0.0.0/0 → Internet Gateway`.
- Cho phép:
  - Amplify/Internet truy cập PostgreSQL OLTP theo SG.

**Private subnet – Analytics**

- Chứa:
  - **EC2 Shiny + DWH (12)**.
  - **Lambda ETL (11)**.
- Không có public IP.
- Chỉ đi ra ngoài qua các **VPC Endpoint (10, 15)**.

**Internet Gateway**

- Cấp đường Internet cho tài nguyên trong public subnet.
- Private subnet không route ra IGW (trừ khi bạn cấu hình khác).

---

### (10) Gateway Endpoint – S3 Gateway Endpoint

**Nhiệm vụ**

- Cho phép tài nguyên trong VPC (Lambda ETL, EC2 private) truy cập S3 (8) **mà không cần NAT/Internet**.
- Traffic đến S3 đi qua **mạng riêng AWS**, bảo mật và rẻ hơn NAT Gateway.

**Vai trò**

- Là mảnh ghép quan trọng để:
  - Giữ nguyên quyết định “**không dùng NAT Gateway**”.
  - Vẫn cho phép ETL / DW đọc ghi S3.

---

### (11) AWS Lambda – ETL

**Nhiệm vụ**

- Được **EventBridge (9)** trigger theo lịch.
- Chạy bên trong **private subnet** của VPC.
- Thực hiện ETL batch:

  - Đọc raw events từ **S3 (8)** qua Gateway Endpoint (10).
  - Transform / clean / flatten theo schema Data Warehouse (ví dụ 15 field bạn đã chốt).
  - Load vào **PostgreSQL DWH trên EC2 private (12)**:
    - Insert / upsert, có thể partition theo ngày/giờ.

- Ghi log và metrics ra **CloudWatch (17)**; khi lỗi có thể gửi thông báo qua **SNS (18)**.

---

### (12) Amazon EC2 – Private (Shiny + DWH)

**Nhiệm vụ**

- Máy chủ **Data Warehouse (PostgreSQL)** + **Shiny Server**.
- Nhận dữ liệu đã chuẩn hoá từ Lambda ETL (11) và lưu vào các bảng DWH.
- Cung cấp dữ liệu cho dashboard Shiny:
  - Shiny truy vấn Postgres local/private.

**Đặc điểm**

- Chạy trong **private subnet**.
- Không có public IP.
- Truy cập quản trị qua **SSM Session Manager (14 + 15)**.

---

### (13) R Shiny Server (trên EC2 Private)

**Nhiệm vụ**

- Host các dashboard phân tích:

  - KPI tổng quan.
  - Conversion funnel.
  - Top products.
  - Hành vi user, session analysis…

- Kết nối trực tiếp tới **PostgreSQL DWH** trên cùng EC2.

**Truy cập**

- Lắng nghe trên port `3838`.
- Thường được truy cập qua:

  - VPN nội bộ, hoặc
  - **SSM port forwarding** (14).

---

### (14) AWS Systems Manager – Session Manager

**Nhiệm vụ**

- Cho admin truy cập EC2 private **không cần SSH, không cần public IP**.
- Hỗ trợ:

  - Mở terminal vào EC2 (run command).
  - **Port forwarding** (ví dụ forward `localhost:3838` → Shiny trên EC2).
  - Audit phiên truy cập tốt hơn.

**Ví dụ**

- Bạn có thể chạy Shiny từ máy local tại:  
  `http://localhost:3838/sbw_dashboard`  
  sau khi cấu hình session port-forward thành công.

---

### (15) Interface Endpoint – SSM VPC Endpoints

**Nhiệm vụ**

- Tạo đường kết nối **private** từ EC2/Lambda trong VPC đến các dịch vụ SSM/Session Manager:

  - `ssm`, `ssmmessages`, `ec2messages`, …

- Đảm bảo:

  - EC2 private vẫn dùng được SSM.
  - Không phải đi qua Internet/NAT.

---

## 5.2.4 Cross-cutting Services

### (16) Amazon IAM

**Nhiệm vụ**

- Quản lý **role/policy** cho toàn hệ thống:

  - Permission cho API Gateway, Lambda.
  - Lambda Ingest ghi S3 (8).
  - Lambda ETL đọc S3 (8) + connect DB (12).
  - EC2 role cho SSM + CloudWatch agent (nếu có).

**Nguyên tắc**

- **Least privilege**.
- Tách quyền rõ theo chức năng và theo từng service.

---

### (17) Amazon CloudWatch

**Nhiệm vụ**

- Thu thập **logs & metrics** từ:

  - Lambda Ingest (7).
  - Lambda ETL (11).
  - EventBridge rule (9).
  - EC2 nếu cài agent.

- Định nghĩa **alarms**: lỗi ETL, lỗi ingest, số lần retry, duration, v.v.

**Vai trò**

- Là nơi **debug chính** khi pipeline có vấn đề.

---

### (18) Amazon SNS

**Nhiệm vụ**

- Là **kênh thông báo** khi có sự cố:

  - ETL fail.
  - Ingest fail.
  - CloudWatch alarm kích hoạt.

- Gửi email / SMS / webhook tuỳ cấu hình.

---

### (20) Amazon EC2 – Public (OLTP)

**Nhiệm vụ**

- Chạy **PostgreSQL OLTP** phục vụ hệ thống web thương mại điện tử:

  - Giao dịch đơn hàng.
  - Thông tin sản phẩm, tồn kho.
  - Thông tin người dùng.

**Liên hệ với DWH**

- **Tách** khỏi Data Warehouse (12):

  - OLTP tối ưu cho **giao dịch**.
  - DWH tối ưu cho **phân tích, batch load**.
