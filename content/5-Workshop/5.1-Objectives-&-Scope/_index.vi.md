---
title: "Objectives & Scope"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Phần này xác định **mục đích**, **mục tiêu học tập** và **phạm vi** của workshop được xây dựng xoay quanh một Nền tảng phân tích clickstream theo lô (batch-based Clickstream Analytics Platform) cho một website thương mại điện tử bán các sản phẩm máy tính.

Nền tảng được triển khai trên AWS và sử dụng:

- Next.js trên **AWS Amplify Hosting** (front-end, Server-Side Rendering).  
- **Amazon CloudFront** (phân phối nội dung toàn cầu).  
- **Amazon Cognito** (xác thực người dùng).  
- **Amazon API Gateway** và **AWS Lambda** (ingest clickstream và ETL).  
- **Amazon S3** (lưu trữ dữ liệu clickstream thô).  
- **Amazon VPC** với các public subnet và private subnet (cô lập mạng).  
- **PostgreSQL** trên **Amazon EC2** (OLTP và Data Warehouse).  
- **R Shiny Server** (các dashboard phân tích).

---

### Bối cảnh nghiệp vụ (Business Context)

Hệ thống mục tiêu là một website thương mại điện tử bán các sản phẩm liên quan đến máy tính (laptop, màn hình, phụ kiện, v.v.).

Doanh nghiệp mong muốn:

- Hiểu **cách người dùng tương tác với website** (các trang đã truy cập, sản phẩm đã xem, sự kiện thêm vào giỏ hàng, các lần thử thanh toán).  
- Đo lường **các phễu chuyển đổi (conversion funnel)** (từ xem sản phẩm đến mua hàng).  
- Xác định **các sản phẩm bán chạy nhất** và các khoảng thời gian có hoạt động cao.  
- Thực hiện được những điều trên **mà không ảnh hưởng** đến cơ sở dữ liệu vận hành và **không phơi bày** các thành phần phân tích nội bộ ra Internet công cộng.

Vì vậy, nền tảng sẽ tách biệt **xử lý giao dịch trực tuyến (OLTP)** khỏi **phân tích**, và thu thập dữ liệu clickstream trong một môi trường phân tích chuyên biệt.

---

### Mục tiêu học tập (Learning Objectives)

Sau khi hoàn thành tất cả các phần (5.1–5.6), người đọc sẽ có thể:

#### Hiểu biết về kiến trúc

- Mô tả được kiến trúc tổng thể của một **nền tảng phân tích clickstream theo lô** trên AWS.  
- Giải thích sự khác nhau giữa:
  - **Miền hướng tới người dùng (user-facing domain)** (Amplify, CloudFront, Cognito, EC2 OLTP).  
  - **Miền ingest & data lake** (API Gateway, Lambda Ingest, S3 Raw bucket).  
  - **Miền phân tích & data warehouse** (ETL Lambda, PostgreSQL DW, Shiny).  
- Lý giải được vì sao OLTP và Analytics được **tách biệt cả về logic lẫn hạ tầng vật lý**, và điều này giúp giảm rủi ro cho workload vận hành như thế nào.

#### Kỹ năng thực hành

- Kích hoạt và kiểm tra luồng ingest clickstream từ frontend vào **API Gateway → Lambda Ingest → S3**.  
- Cấu hình **Gateway VPC Endpoint cho S3** và cập nhật route table để các thành phần phân tích bên trong private subnet có thể truy cập S3 **mà không cần dùng NAT Gateway**.  
- Cấu hình và kiểm thử một **ETL Lambda chạy trong VPC** với khả năng:
  - Đọc các file JSON thô từ S3 Raw Clickstream bucket.  
  - Biến đổi (transform) event thành các bảng phân tích sẵn sàng cho SQL.  
  - Nạp dữ liệu vào PostgreSQL Data Warehouse đặt trên EC2 trong private subnet.  
- Kết nối đến Data Warehouse và chạy các truy vấn **SQL mẫu** để kiểm tra pipeline (đếm số event, top sản phẩm, v.v.).  
- Truy cập các **dashboard R Shiny** chạy trên cùng EC2 với Data Warehouse và diễn giải các biểu đồ chính (funnel, top sản phẩm, chuỗi thời gian).

#### Nhận thức về bảo mật và chi phí

- Giải thích cách **Gateway VPC Endpoint** giữ traffic S3 trên **mạng riêng của AWS**.  
- So sánh việc sử dụng Gateway VPC Endpoint với **NAT Gateway** về **bảo mật**, **chi phí** và **độ phức tạp vận hành**.  
- Liệt kê các cơ chế bảo mật chính được sử dụng trong kiến trúc:
  - Phân tách public subnet và private subnet.  
  - Security group giữa OLTP, ETL Lambda và Data Warehouse.  
  - Quyền hạn IAM tối thiểu cần thiết cho Lambda Ingest và ETL Lambda.

---

### Phạm vi của Workshop (Scope of the Workshop)

Workshop tập trung vào **ba nhóm khả năng cốt lõi** của nền tảng:

1. **Triển khai luồng ingest clickstream**  
   - Ghi nhận tương tác của người dùng trong trình duyệt.  
   - Gửi event dạng JSON đến API Gateway.  
   - Lưu trữ event thô thành các file được phân vùng theo thời gian trong S3 Raw Clickstream bucket.

2. **Xây dựng lớp phân tích riêng tư (private analytics layer)**  
   - Cấu hình VPC, private subnet và S3 Gateway VPC Endpoint.  
   - Chạy ETL Lambda bên trong VPC để Lambda có thể:
     - Đọc từ S3 qua kết nối riêng tư.  
     - Ghi dữ liệu vào PostgreSQL Data Warehouse trên EC2 private.

3. **Trực quan hóa phân tích với các dashboard Shiny**  
   - Truy vấn các bảng phân tích từ R Shiny Server chạy trên cùng EC2 với Data Warehouse.  
   - Cung cấp các dashboard tương tác để phân tích funnel, hiệu suất sản phẩm và xu hướng theo thời gian.

Workshop giả định rằng:

- Phần hạ tầng nền tảng (VPC, subnet, EC2 instance, IAM role, khung Lambda cơ bản, và S3 bucket) đã được provision sẵn, ví dụ thông qua **Terraform** hoặc **CloudFormation**.  
- Trọng tâm là **hiểu và kiểm chứng** luồng dữ liệu và kết nối riêng tư, hơn là viết mã ETL hay mã frontend đạt chuẩn production từ đầu.

---

### Các chủ đề ngoài phạm vi (Out-of-Scope Topics)

Để nội dung tập trung và có thể hoàn thành trong thời gian giới hạn, workshop **không** đề cập đến:

- Các pipeline streaming thời gian thực (ví dụ Amazon Kinesis, Kafka hoặc các dịch vụ streaming được quản lý).  
- Các công nghệ data warehouse nâng cao như **Amazon Redshift**, **Redshift Serverless** hoặc kiến trúc **Lakehouse**.  
- Các mô hình machine learning phức tạp hoặc mô hình phân đoạn người dùng (user segmentation) xây trên dữ liệu clickstream.  
- Pipeline CI/CD ở mức production, triển khai blue/green, hay mô hình nhiều tài khoản AWS (multi-account).  
- Tối ưu hóa sâu truy vấn SQL và thiết kế index vượt ngoài các ví dụ đơn giản.

Các chủ đề này được xem là **phần mở rộng trong tương lai** của nền tảng và có thể được khám phá trong các công việc tiếp theo.

---

### Đối tượng mục tiêu (Target Audience)

Workshop chủ yếu dành cho:

- Sinh viên hoặc kỹ sư đã có hiểu biết cơ bản về AWS và muốn thấy cách nhiều dịch vụ kết hợp với nhau thành một giải pháp phân tích thực tế.  
- Lập trình viên quen thuộc với **JavaScript/TypeScript**, **SQL** và môi trường **Linux**.  
- Bất kỳ ai quan tâm đến việc thiết kế một kiến trúc phân tích **an toàn, tiết kiệm chi phí**, chủ yếu dựa trên các dịch vụ serverless và một footprint EC2 nhỏ.

Người đọc không cần phải là chuyên gia về mạng hoặc data engineering; các phần sau (5.2–5.5) sẽ cung cấp các bước cụ thể, có hướng dẫn rõ ràng để giúp kiến trúc trở nên trực quan và có thể tái tạo.

**Hình 5-1: Amplify hosting cho frontend thương mại điện tử**

Ảnh chụp màn hình thể hiện ứng dụng Amplify đang host frontend Next.js của website thương mại điện tử, bao gồm nhánh chính (main branch), trạng thái lần triển khai gần nhất và URL CloudFront được sử dụng trong workshop này.

![Hình 5-1: Amplify hosting cho frontend thương mại điện tử](/images/5-a-amplify-overview.png)
