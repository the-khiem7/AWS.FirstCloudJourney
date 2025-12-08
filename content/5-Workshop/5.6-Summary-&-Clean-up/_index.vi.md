---
title: "Summary & Clean up"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Phần cuối cùng này sẽ:

- Tóm tắt lại các khái niệm và thành phần chính mà bạn đã làm việc xuyên suốt workshop.  
- Nhấn mạnh cách chúng kết hợp với nhau thành một **Nền tảng phân tích Clickstream theo lô (Batch-based Clickstream Analytics Platform)** hoàn chỉnh.  
- Cung cấp một **checklist dọn dẹp (clean-up)** để bạn có thể dừng hoặc xoá các tài nguyên AWS và tránh chi phí không cần thiết sau khi kết thúc thực hành.

---

### Tóm tắt những điểm chính đã học

Xuyên suốt các Mục **5.1–5.5**, bạn đã xây dựng và kiểm chứng một pipeline phân tích đầu–cuối cho một website thương mại điện tử bán các sản phẩm máy tính.

#### Kiến trúc và thiết kế

Từ **5.1 Objectives & Scope** và **5.2 Architecture Walkthrough**, bạn đã học cách:

- Mô tả các khối chức năng cốt lõi của nền tảng:

  - **Frontend & miền hướng tới người dùng (user-facing domain)**:  
    - Ứng dụng Next.js trên **AWS Amplify Hosting**.  
    - Phân phối qua **Amazon CloudFront**.  
    - Xác thực người dùng bằng **Amazon Cognito**.

  - **Miền ingest & data lake**:  
    - **Amazon API Gateway (HTTP API)** (`clickstream-http-api`) làm endpoint ingest clickstream.  
    - **Lambda Ingest** (`clickstream-lambda-ingest`) để validate và enrich event.  
    - **S3 Raw Clickstream bucket** (`clickstream-s3-ingest`) lưu trữ log JSON được phân vùng theo thời gian.

  - **Miền Analytics & Data Warehouse**:  
    - **VPC (10.0.0.0/16)** với **public OLTP subnet (10.0.0.0/20)** và **private Analytics & ETL subnet (10.0.128.0/20)**.  
    - **PostgreSQL OLTP trên EC2** (`SBW_EC2_WebDB`) trong public subnet.  
    - **PostgreSQL Data Warehouse trên EC2** (`SBW_EC2_ShinyDWH`) trong private subnet.  
    - **R Shiny Server** cùng EC2 đó để hiển thị dashboard phân tích.  
    - **Lambda ETL có gắn VPC** (`SBW_Lamda_ETL`) trong private subnet.  
    - **AWS Systems Manager Session Manager** với VPC Interface Endpoints cho truy cập admin an toàn, không cần SSH vào các private instance.

- Giải thích vì sao nền tảng tách biệt:

  - Workload **OLTP (Operational)** (đơn hàng, tồn kho, người dùng thời gian thực) và  
  - Workload **Analytics (Read-heavy)** (phân tích funnel, hiệu suất sản phẩm, chuỗi thời gian).

Việc tách biệt này cải thiện hiệu năng, độ ổn định và bảo mật cho hệ thống.

##### Đường ingest: Frontend → API Gateway → S3

Từ **5.3 Implementing Clickstream Ingestion**, bạn đã:

- Triển khai và/hoặc hiểu một **luồng tracking** trong đó:

  - Trình duyệt thu thập các event page view, product view, add-to-cart và checkout.  
  - Event được tuần tự hoá thành payload JSON chứa metadata về người dùng, session và sản phẩm.  
  - Frontend gọi `POST /clickstream` trên **API Gateway**.

- Thấy được cách **Lambda Ingest**:

  - Parse các event nhận được.  
  - Thực thi validate tối thiểu và gắn thêm metadata phía server.  
  - Ghi các object JSON dạng batch vào **S3 Raw Clickstream bucket** sử dụng prefix phân vùng, ví dụ:

    ```text
    events/YYYY/MM/DD/HH/events-<uuid>.json
    ```

- Thực hành **kiểm thử thủ công** bằng cả:

  - Frontend thật (thông qua Developer Tools của trình duyệt), và  
  - Các API client như Postman hoặc Thunder Client.

#### Lớp phân tích riêng tư: Gateway Endpoint, Lambda ETL, Data Warehouse

Từ **5.4 Building the Private Analytics Layer**, bạn đã:

- Cấu hình **S3 Gateway VPC Endpoint**, đảm bảo rằng:

  - Private subnet (10.0.128.0/20) **không** cần public IP hoặc NAT Gateway để truy cập S3.  
  - Traffic giữa Lambda ETL và S3 luôn chạy trên **mạng riêng nội bộ của AWS**.

- Gắn **Lambda ETL** vào VPC bằng cách:

  - Chọn ETL private subnet và security group phù hợp.  
  - Cấp cho IAM execution role của Lambda các quyền tối thiểu để đọc từ Raw bucket và ghi log vào CloudWatch.

- Cài đặt logic ETL để:

  - Liệt kê các object S3 trong một khoảng thời gian nhất định.  
  - Parse các event clickstream JSON thô.  
  - Transform chúng thành các cấu trúc quan hệ (ví dụ: `fact_events`, `dim_products`).  
  - Nạp dữ liệu vào **PostgreSQL Data Warehouse** trên EC2 trong Analytics private subnet.

- Kiểm chứng kết nối và bảo mật thông qua:

  - Security group chỉ cho phép **Lambda ETL** kết nối tới Data Warehouse trên port `5432`.  
  - Route table trong private subnet chỉ chứa các route `local` + prefix-list tới S3, **không có 0.0.0.0/0**.

#### Phân tích và trực quan hoá với Shiny

Từ **5.5 Visualizing Analytics with Shiny Dashboards**, bạn đã:

- Trigger job ETL (qua EventBridge hoặc thủ công) và kiểm tra **tính mới** của dữ liệu bằng các truy vấn SQL:

  - Đếm tổng số event.  
  - Phân bố event theo loại.  
  - Top sản phẩm được xem nhiều nhất.  
  - Các chỉ số funnel đơn giản.

- Truy cập **R Shiny dashboards** chạy trên EC2 private cùng với Data Warehouse bằng cách sử dụng **AWS Systems Manager Session Manager port forwarding**:

  - Không cần SSH key hay bastion host.  
  - Tunnel hoàn toàn mã hóa qua HTTPS thông qua VPC Interface Endpoints.  
  - Kiểm soát truy cập qua IAM với đầy đủ audit trail.

- Khám phá các dashboard mô tả:

  - **Phễu và hành trình người dùng** (page view → product view → add-to-cart → checkout → purchase).  
  - **Hiệu suất sản phẩm** (top sản phẩm theo lượt xem và đơn hàng).  
  - **Xu hướng theo thời gian** (số event theo giờ/ngày).

- Kết nối các biểu đồ trong Shiny với các **truy vấn SQL và bảng trong Data Warehouse**, xác nhận rằng:

  - Dashboard nhất quán với dữ liệu.  
  - Bạn có thể kiểm tra lại các chỉ số quan trọng bằng SQL khi cần.

---

### Checklist dọn dẹp (Clean-up) các tài nguyên AWS

Sau khi hoàn thành workshop, nên **dọn dẹp tài nguyên** để tránh chi phí phát sinh. Cách xử lý cụ thể tuỳ thuộc vào việc môi trường này là:

- Một môi trường lab/demo ngắn hạn, hay  
- Một môi trường phát triển/staging cần giữ lại lâu dài.

> **Quan trọng:** Trước khi xoá bất kỳ thứ gì, hãy đảm bảo rằng **không có ai khác** đang sử dụng cùng AWS account hoặc các tài nguyên chung.

#### EC2 instances

1. **EC2 OLTP (`SBW_EC2_WebDB`, Public Subnet)**

   - Nếu không còn cần cơ sở dữ liệu vận hành:  
     - Dừng instance để ngưng chi phí compute, hoặc  
     - Terminate để xoá vĩnh viễn.  
   - Trước khi terminate, cân nhắc:  
     - Chụp snapshot volume hoặc backup cuối.  
     - Dùng `pg_dump` hoặc công cụ tương tự để xuất dữ liệu quan trọng.

2. **EC2 Data Warehouse + Shiny (`SBW_EC2_ShinyDWH`, Private Subnet)**

   - Nếu chỉ dùng cho workshop:  
     - Dừng hoặc terminate sau khi đã export kết quả phân tích hoặc schema cần giữ.  
   - Nếu dự định mở rộng lớp analytics sau này:  
     - Có thể giữ instance nhưng cần:  
       - Gỡ bỏ các service không dùng (các app Shiny thử nghiệm).  
       - Đảm bảo security group vẫn chặt chẽ (không inbound rộng).

#### Lambda functions và EventBridge rules

1. **Lambda Ingest (`clickstream-lambda-ingest`)**

   - Nếu bạn không còn gửi clickstream event:  
     - Có thể xoá function để giao diện console gọn gàng hơn.  
   - Nếu vẫn muốn dùng để thử nghiệm tiếp:  
     - Giữ lại, nhưng cân nhắc tạm thời tắt frontend hoặc sửa cấu hình.

2. **Lambda ETL (`SBW_Lamda_ETL`)**

   - Nếu EC2 Data Warehouse đã dừng hoặc xoá:  
     - Disable hoặc xoá Lambda ETL (và EventBridge rule đi kèm) để tránh các lần invoke thất bại.

3. **EventBridge ETL schedule (`SBW_ETL_HOURLY_RULE`)**

   - Vào **Amazon EventBridge → Rules**.  
   - Disable hoặc xoá rule (`SBW_ETL_HOURLY_RULE`) để nó không kích hoạt ETL nữa.

#### S3 buckets và dữ liệu

1. **Raw Clickstream S3 bucket (`clickstream-s3-ingest`)**

   - Quyết định có giữ lại dữ liệu raw hay không:

     - Với **lab ngắn hạn**, bạn có thể xoá các prefix `events/YYYY/MM/DD/` được tạo trong lúc test.  
     - Với **dự án dài hạn**, giữ lại bucket nhưng:  
       - Cân nhắc bật **S3 lifecycle policy** để:  
         - Chuyển dữ liệu cũ sang storage class rẻ hơn, hoặc  
         - Xoá sau N ngày.

2. **Các bucket khác cho project**

   - Nếu có các bucket chỉ phục vụ workshop (log, artifact, screenshots…), hãy rà soát và xoá nếu không cần nữa.

> **Lưu ý:** Chi phí S3 có thể tăng dần theo thời gian; nên định kỳ kiểm tra nội dung bucket và rule lifecycle.

#### VPC Endpoint và thành phần mạng

1. **S3 Gateway VPC Endpoint**

   - Nếu VPC không còn dùng cho analytics:  
     - Có thể xoá **Gateway Endpoint**.  
   - Nếu VPC vẫn dùng chung cho các workload khác:  
     - Có thể giữ lại để dùng tiếp.

2. **SSM Interface VPC Endpoints**

   - Xoá các VPC Interface Endpoints cho AWS Systems Manager nếu không còn cần:  
     - `com.amazonaws.<region>.ssm`  
     - `com.amazonaws.<region>.ssmmessages`  
     - `com.amazonaws.<region>.ec2messages`  
   - Vào **VPC** → **Endpoints**, chọn từng endpoint và bấm **Actions** → **Delete endpoint**.

   > **Lưu ý**: Interface Endpoints tính phí theo giờ (~$0.01/giờ mỗi endpoint mỗi AZ). Gateway Endpoints cho S3 miễn phí.

3. **Route table, subnet, VPC**

   - Với VPC dành riêng cho lab:  
     - Có thể xoá toàn bộ VPC (xoá luôn subnet, route table, endpoint đi kèm) **sau khi** chắc chắn không còn EC2, RDS hay tài nguyên quan trọng nào phụ thuộc.  
   - Với VPC dùng chung:  
     - Chỉ xoá các subnet/route table tạo riêng cho workshop, tránh ảnh hưởng môi trường khác.

#### CloudWatch Logs và monitoring

1. **CloudWatch Logs cho Lambda và API Gateway**

   - Log group có thể lớn dần theo thời gian.  
   - Đặt **retention policy** phù hợp (7, 30, 90 ngày) cho:  
     - Lambda Ingest.  
     - Lambda ETL.  
     - API Gateway access log.

2. **CloudWatch Alarms và metric**

   - Nếu bạn tạo các alarm riêng cho môi trường này (ví dụ: alarm lỗi ingest hoặc ETL):  
     - Disable hoặc xoá khi decommission môi trường.

**Hình 5-14: CloudWatch alarms cho nền tảng clickstream**

Ảnh chụp màn hình hiển thị một tập các CloudWatch alarms cho đường ingest và Data Warehouse, bao gồm:

- Alarm latency và tỉ lệ 4xx/5xx của API.  
- Alarm duration và throttling cho Lambda Ingest.  
- Alarm status check và CPU utilization cho EC2 Data Warehouse.

Phần lớn alarm ở trạng thái `OK` hoặc `Insufficient data`, cho thấy hệ thống đang “khỏe mạnh” và có thể được tắt/xoá an toàn sau workshop.

![Hình 5-14: CloudWatch alarms cho nền tảng clickstream](/images/5-6-cloudwatch-alarms.png)

#### IAM roles và policies

1. **Lambda execution roles**

   - Rà soát các IAM role tạo cho:  
     - Lambda Ingest (`clickstream-lambda-ingest`).  
     - Lambda ETL (`SBW_Lamda_ETL`).  

   - Nếu không còn cần:  
     - Gỡ các policy đính kèm.  
     - Xoá role để tránh tích luỹ các IAM identity không dùng đến.

2. **User/role test**

   - Nếu bạn tạo IAM user hoặc role chỉ để test trong workshop:  
     - Xoá hoặc siết chặt permission.

#### Amplify, Cognito và API Gateway

1. **Amplify app**

   - Nếu frontend Next.js chỉ dùng cho workshop:  
     - Xoá **Amplify app** và các branch/environment tương ứng.

2. **Cognito User Pool**

   - Xoá các user test hoặc xoá toàn bộ User Pool nếu nó chỉ phục vụ workshop.

3. **API Gateway HTTP API (`clickstream-http-api`)**

   - Xoá API ingest clickstream nếu không còn dùng, nhất là khi nó được tạo riêng cho lab.

---

### Nên giữ lại những gì cho các bước kế tiếp?

Nếu bạn dự định **mở rộng** project sau workshop, hãy cân nhắc giữ lại:

- **Code repository**:

  - Frontend Next.js (bao gồm tracking logic).  
  - Lambda Ingest và Lambda ETL.  
  - Mã Shiny app.  
  - File hạ tầng như Terraform/CloudFormation.

- **Schema Data Warehouse và một tập dữ liệu mẫu nhỏ**:

  - Một EC2 instance nhỏ với database chứa vài nghìn event là đủ để tiếp tục phát triển dashboard và truy vấn mà không cần ingest volume lớn.

- **Sơ đồ kiến trúc và ghi chú**:

  - Các sơ đồ dùng trong Hình 5-1 đến 5-6 (và các hình ở phần 5.3–5.5).  
  - Lý do thiết kế (ví dụ: dùng Gateway Endpoint thay vì NAT Gateway).

Những tài sản này sẽ giúp bạn tiếp tục project hoặc chuyển đổi nó thành giải pháp nâng cao hơn (ví dụ: chuyển sang **Amazon Redshift Serverless** hoặc thêm **Streaming real-time**).

---

### Lời kết

Workshop này đã minh hoạ cách:

- Thiết kế và triển khai một kiến trúc phân tích **an toàn, tiết kiệm chi phí và dễ mở rộng** trên AWS.  
- Thu thập event clickstream từ một frontend thương mại điện tử thực tế.  
- Xây dựng một quy trình ETL riêng tư đọc từ S3 và nạp dữ liệu vào một Data Warehouse riêng.  
- Trực quan hoá hành vi người dùng và hiệu suất sản phẩm bằng R Shiny, **mà không cần phơi bày backend analytics ra Internet công cộng**.

Giờ đây bạn đã có một “bản mẫu” (reference) cho việc xây dựng các nền tảng phân tích tương tự, cùng một nền tảng kiến thức vững chắc để thực hiện các nâng cấp trong tương lai như:

- Streaming real-time (Kinesis / Kafka).  
- Mô hình phân bổ (attribution) nâng cao và phân đoạn người dùng.  
- Di chuyển Data Warehouse sang các dịch vụ quản lý như Amazon Redshift.

---

### Hình 5-15: Danh sách tài nguyên cần rà soát sau workshop

Sơ đồ hoặc bảng trong hình này nên liệt kê, với mỗi dịch vụ AWS:

- Loại tài nguyên (EC2, S3, Lambda, EventBridge, API Gateway, VPC Endpoint, …).  
- Ví dụ tên tài nguyên cụ thể.  
- Hành động khuyến nghị sau workshop (Keep / Stop / Delete / Set retention).

![Hình 5-15: Các tài nguyên chính cần kiểm tra và dọn dẹp sau workshop](/images/5-7-cleanup.png)
