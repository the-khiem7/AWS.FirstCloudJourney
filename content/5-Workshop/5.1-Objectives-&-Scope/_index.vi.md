---
title: "Objectives & Scope"
weight: 51
chapter: false
pre: " <b> 5.1. </b> "
---

### Bối cảnh kinh doanh

Hệ thống mục tiêu là một website e-commerce bán laptop, màn hình và phụ kiện.  
Business muốn:

- Hiểu **cách người dùng tương tác** với website:
  - Họ truy cập những trang nào  
  - Xem sản phẩm nào  
  - Các sự kiện add-to-cart, checkout  
- Đo lường **conversion funnel** từ product view → add to cart → checkout  
- Xác định:
  - Sản phẩm top (bán chạy / được xem nhiều)  
  - Khung giờ hoạt động cao điểm (theo giờ, theo ngày)  
- Giúp sử dụng chiến lược marketing phù hợp nhất để tăng doanh thu, tiết kiệm chi phí
  
Để đạt được điều đó, nền tảng:

- Thu thập bộ dữ liệu hành vi người dùng bằng clickstream, sau đó đưa ra thống kê và phân tích

---

### Mục tiêu học tập

#### Hiểu kiến trúc

- Giải thích được kiến trúc tổng thể của một **batch-based clickstream analytics platform** sử dụng:
  - Amplify, CloudFront, Cognito, EC2 OLTP ở **miền user-facing**  
  - API Gateway, Lambda Ingest, S3 Raw bucket ở **miền ingestion & data lake**  
  - ETL Lambda, PostgreSQL DW trên EC2, R Shiny ở **miền analytics & DW**  
- Giải thích được vì sao OLTP và Analytics được tách:
  - **Logic**: khác schema, khác loại workload  
  - **Physical**: public subnet vs private subnet, 2 EC2 khác nhau  

#### Kỹ năng thực hành

- Gửi clickstream events từ frontend vào API Gateway → Lambda Ingest → S3 Raw (`clickstream-s3-ingest`).  
- Cấu hình **Gateway VPC Endpoint cho S3** và chỉnh route table private để các thành phần private truy cập S3.  
- Cấu hình và test một **ETL Lambda** (`SBW_Lamda_ETL`) có khả năng:
  - Đọc file JSON thô từ `s3://clickstream-s3-ingest/events/YYYY/MM/DD/`  
  - Transform events thành các dòng dữ liệu cho bảng `clickstream_dw.public.clickstream_events`  
- Kết nối vào DW (`SBW_EC2_ShinyDWH`) và chạy các câu truy vấn SQL mẫu:
  - Đếm số lượng events  
  - Top sản phẩm  
  - Funnel cơ bản  
- Truy cập **R Shiny dashboards** qua SSM port forwarding và đọc hiểu:
  - Biểu đồ funnel  
  - Biểu đồ về mức độ tương tác sản phẩm  
  - Biểu đồ time-series hoạt động theo thời gian  

#### Nhận thức về bảo mật và chi phí

- Hiểu được tầm quan trọng của việc bảo mật dữ liệu hành vi người dùng bằng cách không cho EC2_ShinyDWH, Lamda_ETL bằng cách cho EC2_ShinyDWH, Lamda_ETL vào Private subnet. 
  - Chỉ cho Lamda_ETL truy cập S3 qua **Gateway VPC Endpoint**  
  - Chỉ cho admin đi vào EC2_ShinyDWH thông qua SSM bằng **Interface VPC Endpoints**  
- Nhận biết các control bảo mật chính:
  - Phân tách public subnet và private subnet  
  - Security group giữa `sg_oltp_webDB`, `sg_Lambda_ETL`, `sg_analytics_ShinyDWH`  
  - IAM permission tối thiểu cho từng Lambda  
  - Zero-SSH admin sử dụng AWS Systems Manager Session Manager (không cần bastion host, không mở cổng SSH).

---

### Phạm vi Workshop

Workshop tập trung vào 3 nhóm khả năng chính:

1. **Triển khai lớp ingestion clickstream**
   - Thu thập tương tác trên browser trong frontend Next.js  
   - Gửi sự kiện dạng JSON tới API Gateway (`clickstream-http-api`)  
   - Lưu events thô theo thời gian trong `clickstream-s3-ingest`  

2. **Xây dựng lớp analytics private**
   - Tạo VPC, subnets, route tables và VPC endpoints  
   - Chạy `SBW_Lamda_ETL` bên trong VPC  
   - Kết nối Lambda ETL tới EC2 Data Warehouse private (`SBW_EC2_ShinyDWH`)  

3. **Trực quan hóa analytics với Shiny dashboards**
   - Truy vấn `clickstream_dw` từ R Shiny  
   - Hiển thị funnel, hiệu quả sản phẩm và trend theo thời gian  
   - Truy cập Shiny thông qua **SSM Session Manager port forwarding**  

---

### Ngoài phạm vi

Để workshop không quá phức tạp, chúng ta **không** đi sâu vào:

- Real-time streaming (Kinesis, Kafka, MSK, …)  
- Dịch vụ DW nâng cao (Amazon Redshift / Redshift Serverless)  
- Khuyến nghị, segmentation hay anomaly detection dùng ML  
- CI/CD production, blue/green deployment, multi-account  
- Tối ưu SQL nâng cao, thiết kế index chi tiết  

Đây là những hướng mở rộng tự nhiên sau khi nền tảng batch-based clickstream đã được dựng xong.

 


