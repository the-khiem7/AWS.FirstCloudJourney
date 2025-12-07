---
title: "Building the Private Analytics Layer"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Phần này giải thích chi tiết cách triển khai **lớp phân tích riêng tư (private analytics layer)** để toàn bộ ETL theo lô có thể chạy **hoàn toàn bên trong các private subnet**, mà không cần phơi bày các thành phần nội bộ ra Internet công cộng.

Lớp phân tích riêng tư bao gồm:

- Một **Lambda ETL có VPC (VPC-enabled ETL Lambda)** chạy trong **private subnet**.  
- Một **S3 Gateway VPC Endpoint** cho phép Lambda ETL truy cập S3 qua **mạng riêng AWS**.  
- Một **PostgreSQL Data Warehouse** chạy trên **EC2 instance** trong một private subnet riêng.

Những thành phần này kết hợp lại tạo thành xương sống của pipeline xử lý theo lô:

> S3 Raw Clickstream bucket → ETL Lambda (trong VPC) → PostgreSQL Data Warehouse (EC2, private subnet)

---

### Thiết kế mạng cho lớp phân tích (Networking design for the analytics layer)

VPC được chia logic thành các subnet với vai trò khác nhau:

- **Public Subnet – OLTP (10.0.1.0/24)**  
  - Chứa EC2 PostgreSQL OLTP.  
  - Có route `0.0.0.0/0 → Internet Gateway (IGW)` để truy cập Internet hai chiều.

- **Private Subnet – Analytics (10.0.2.0/24)**  
  - Chứa **EC2 Data Warehouse** và **R Shiny Server**.  
  - **Không** có route tới Internet Gateway và **không có NAT Gateway**.  
  - Chỉ có các route nội bộ trong VPC và kết nối tới các private subnet khác.

- **Private Subnet – ETL (10.0.3.0/24)**  
  - Chứa **Lambda ETL trong VPC** và **S3 Gateway VPC Endpoint**.  
  - Cũng **không** có route `0.0.0.0/0` và không có NAT Gateway.  
  - Có thể truy cập S3 một cách riêng tư qua Gateway Endpoint.

Thiết kế này đảm bảo rằng:

- Data Warehouse và R Shiny **không thể truy cập trực tiếp từ Internet công cộng**.  
- Lambda ETL chỉ có thể truy cập S3 và Data Warehouse **thông qua mạng riêng của AWS**.  
- **Không cần NAT Gateway**, giúp giảm chi phí và đơn giản hoá cấu trúc mạng.

---

### Tạo và cấu hình S3 Gateway VPC Endpoint

S3 Gateway VPC Endpoint cho phép tài nguyên trong private subnet truy cập S3 **mà không cần dùng public IP**.

**Hình 5-9: S3 Gateway VPC Endpoint gắn với các private route table**

Ảnh chụp màn hình hiển thị Gateway VPC Endpoint cho dịch vụ Amazon S3 (`com.amazonaws.ap-southeast-1.s3`) trong VPC của workshop.  
Endpoint có trạng thái `Available` và được gắn với hai private route table, vốn tương ứng với các subnet Analytics và ETL.  
Cấu hình này đảm bảo traffic từ Lambda ETL và EC2 Data Warehouse tới S3 luôn đi trong mạng nội bộ AWS, không cần NAT Gateway.

![Figure 5-9: Gateway VPC Endpoint attached to private route tables](/images/5-4-s3-gateway-vpce.png)

#### Bước 1 – Tạo Gateway Endpoint

1. Mở **VPC Console** trong AWS Management Console.  
2. Điều hướng tới **Endpoints** → bấm **Create endpoint**.  
3. Ở mục **Service category**, chọn **AWS services**.  
4. Trong ô tìm kiếm, nhập `s3` và chọn mục:

   ```text
   com.amazonaws.<region>.s3
   ```

5. Ở **Endpoint type**, chọn **Gateway**.  
6. Ở **VPC**, chọn VPC của project, nơi chứa các subnet analytics và ETL.  
7. Trong **Route tables**, chọn route table gắn với **ETL private subnet (10.0.3.0/24)** và, nếu muốn, thêm route table của **Analytics private subnet (10.0.2.0/24)** nếu bạn cũng muốn EC2 Data Warehouse truy cập S3 riêng tư.  
8. Ở phần **Policy**, trong workshop có thể bắt đầu với **Full access**:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:*",
         "Resource": "*"
       }
     ]
   }
   ```

   Sau này có thể siết lại để chỉ cho phép một bucket hoặc prefix cụ thể.

9. Bấm **Create endpoint**.

#### Bước 2 – Kiểm tra các route table

Sau khi endpoint được tạo, AWS sẽ tự động thêm route vào các route table đã chọn.

1. Trong **VPC Console**, mở **Route tables**.  
2. Chọn route table dùng cho **ETL private subnet**.  
3. Ở tab **Routes**, kiểm tra:

   - Có route local:

     ```text
     10.0.0.0/16 → local
     ```

   - Có route dùng prefix-list tới S3, trỏ tới endpoint mới:

     ```text
     pl-xxxxxxxx → vpce-xxxxxxxx  (Gateway Endpoint to S3)
     ```

   - **Không** có route:

     ```text
     0.0.0.0/0 → igw-xxxxxxx
     ```

   - **Không** có NAT Gateway.

Như vậy, các private subnet **không** gửi traffic trực tiếp ra Internet, nhưng **vẫn truy cập được S3** qua Gateway Endpoint.

---

### Cấu hình Lambda ETL bên trong VPC

Lambda ETL phải được đặt trong VPC để có thể:

- Truy cập S3 thông qua **Gateway Endpoint**.  
- Kết nối tới **PostgreSQL Data Warehouse** trong Analytics private subnet.

#### Bước 1 – Gắn Lambda vào VPC

1. Mở **Lambda Console** và chọn function ETL, ví dụ:

   ```text
   clickstream-etl-lambda
   ```

2. Vào tab **Configuration** → mục **Network** (hoặc **Environment → VPC** tuỳ UI).  
3. Bấm **Edit** và thiết lập:

   - **VPC**: VPC của project.  
   - **Subnets**: chọn **ETL private subnet (10.0.3.0/24)** (có thể chọn nhiều subnet ở các AZ khác nhau để tăng khả dụng).  
   - **Security groups**: chọn security group:
     - Cho phép outbound tới S3 (qua Gateway Endpoint).  
     - Cho phép outbound tới EC2 Data Warehouse (port 5432).

4. Lưu cấu hình. Ở lần chạy tiếp theo, Lambda sẽ tạo các ENI (elastic network interface) trong subnet đã chọn.

#### Bước 2 – Quyền IAM cho Lambda ETL

**Execution role** của Lambda ETL cần có:

- Quyền **đọc từ** Raw Clickstream S3 bucket, ví dụ:

  ```json
  {
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::clickstream-raw-<account>-<region>",
      "arn:aws:s3:::clickstream-raw-<account>-<region>/events/*"
    ]
  }
  ```

- Quyền ghi log vào **CloudWatch Logs** (thường được thêm sẵn).  
- Không cần các quyền quản trị hạ tầng khác (giữ role ở mức tối thiểu cần thiết).

#### Bước 3 – Cấu hình kết nối cơ sở dữ liệu

Trong phần environment variables của Lambda ETL, lưu các biến:

- `DW_HOST` – private IP hoặc hostname của EC2 Data Warehouse.  
- `DW_PORT` – thường là `5432`.  
- `DW_USER` / `DW_PASSWORD` – tài khoản có quyền insert vào các bảng DW.  
- `DW_DATABASE` – tên database analytics.

Mã Lambda sẽ sử dụng các biến này để tạo kết nối (ví dụ thông qua thư viện client PostgreSQL).

---

### Cài đặt logic ETL

Mỗi lần chạy (được lên lịch qua EventBridge hoặc invoke thủ công), Lambda ETL thực hiện các bước ở mức cao như sau:

1. **Xác định khoảng thời gian / prefix S3 cần xử lý**  
   - Ví dụ: toàn bộ object dưới `events/YYYY/MM/DD/HH/` cho giờ gần nhất.

2. **Liệt kê các object S3 tương ứng**  
   - Dùng `ListObjectsV2` trên Raw Clickstream bucket với prefix đã chọn.

3. **Đọc và parse các event**  
   - Với mỗi object, gọi `GetObject` và parse nội dung JSON.  
   - Validate các trường quan trọng (event name, timestamp, user/session ID, product ID).

4. **Transform sang schema phân tích**  
   - Map JSON thô sang các bảng quan hệ, ví dụ:

     - `dim_users` – thuộc tính ở cấp người dùng.  
     - `dim_products` – thuộc tính ở cấp sản phẩm.  
     - `fact_events` – từng event riêng lẻ, có khoá ngoại tới users và products.  
     - `fact_sessions` – thông tin tổng hợp theo session.

   - Áp dụng các rule nghiệp vụ đơn giản, như:
     - Suy ra **ngày** và **giờ** từ timestamp.  
     - Chuẩn hoá tên event.  
     - Loại bỏ event test hoặc sai format.

5. **Ghi vào Data Warehouse**

   - Mở một transaction tới PostgreSQL Data Warehouse.  
   - Dùng các lệnh `INSERT` dạng batch hoặc cơ chế bulk-load (tuỳ triển khai).  
   - Commit nếu thành công; rollback nếu có lỗi.

6. **Đánh dấu tiến độ (tuỳ chọn)**  
   - Lưu trạng thái đã xử lý (ví dụ: giờ cuối cùng đã ETL) trong một bảng control hoặc một prefix metadata trên S3 để tránh xử lý trùng.

Ví dụ một bảng fact đơn giản cho events:

```sql
CREATE TABLE IF NOT EXISTS fact_events (
    event_id          UUID PRIMARY KEY,
    event_timestamp   TIMESTAMPTZ NOT NULL,
    event_name        TEXT NOT NULL,
    user_id           TEXT,
    session_id        TEXT,
    client_id         TEXT,
    product_id        TEXT,
    page_url          TEXT,
    traffic_source    TEXT,
    created_at        TIMESTAMPTZ DEFAULT NOW()
);
```

Lambda ETL sẽ insert dữ liệu vào `fact_events` dựa trên các file JSON đã đọc từ S3.

**Hình 5-10: Cấu hình VPC cho Lambda ETL**

Ảnh chụp màn hình hiển thị function `SBW_Lamda_ETL` được gắn với `SBW_Project-vpc`.  
Lambda sử dụng hai private subnet (mỗi subnet ở một Availability Zone) và một security group riêng (`sg_Lamda_ETL`) để kiểm soát toàn bộ truy cập mạng.  
Cấu hình này cho phép function truy cập S3 Gateway VPC Endpoint và EC2 Data Warehouse mà không cần phơi bày Lambda ra Internet công cộng.

![Figure 5-10: VPC configuration of the ETL Lambda function](/images/5-4-lambda-etl-vpc.png)

---

### Security group và kết nối (Security groups and connectivity)

Để đảm bảo chỉ cho phép các luồng traffic cần thiết:

- **Security Group cho EC2 Data Warehouse (SG-DW)**

  - Inbound:
    - Cho phép `5432/tcp` **từ security group của Lambda ETL** (không từ `0.0.0.0/0`).  
  - Outbound:
    - Cho phép tất cả outbound (hoặc siết chặt hơn nếu cần cho mục đích update/monitoring).

- **Security Group cho Lambda ETL (SG-ETL)**

  - Inbound:
    - Không cần (Lambda không nhận kết nối inbound).  
  - Outbound:
    - Cho phép traffic tới:
      - Private IP của EC2 Data Warehouse ở port `5432`.  
      - S3 Gateway VPC Endpoint (thường chỉ cần cho phép all outbound trong VPC).

Thiết lập này đảm bảo rằng:

- Chỉ Lambda ETL mới nói chuyện được với Data Warehouse (không có truy cập trực tiếp từ Internet).  
- EC2 Data Warehouse không mở cổng tuỳ ý cho các kết nối inbound khác.

---

### Kiểm thử pipeline ETL end-to-end

Để xác nhận lớp phân tích riêng tư hoạt động đúng:

1. **Sinh các event mới**  
   - Thực hiện lại các bước ở Mục 5.3 để tạo clickstream mới (page view, product view, add-to-cart, checkout).

2. **Trigger Lambda ETL**

   - Cách A: Chờ **EventBridge rule** chạy theo lịch (ví dụ mỗi 30 phút).  
   - Cách B: Trong EventBridge console, chọn rule ETL (ví dụ `clickstream-etl-schedule`) và bấm **Run now**.  
   - Cách C: Trong Lambda console, dùng nút **Test** với một test event tối giản để invoke Lambda ETL thủ công.

3. **Kiểm tra CloudWatch Logs**

   - Mở log group của Lambda ETL trong **CloudWatch Logs**.  
   - Xác minh rằng:
     - Lambda đã liệt kê và đọc các object S3 thành công.  
     - Số lượng event đã xử lý khớp với kỳ vọng.  
     - Kết nối tới Data Warehouse và các lệnh insert được thực thi không lỗi.

4. **Kiểm tra dữ liệu trong Data Warehouse**

   - Kết nối vào EC2 Data Warehouse bằng **Session Manager** hoặc SSH.  
   - Từ đó, sử dụng `psql` hoặc client SQL bất kỳ để chạy các truy vấn ví dụ:

     ```sql
     SELECT event_name, COUNT(*) AS total_events
     FROM fact_events
     GROUP BY event_name
     ORDER BY total_events DESC;
     ```

     ```sql
     SELECT product_id, COUNT(*) AS view_count
     FROM fact_events
     WHERE event_name = 'product_view'
     GROUP BY product_id
     ORDER BY view_count DESC
     LIMIT 10;
     ```

   - Xác nhận rằng số lượng và top sản phẩm khớp tương đối với hành vi mà bạn đã thực hiện trên website.

---

### Tóm tắt

Sau phần này, bạn đã:

- Cấu hình **S3 Gateway VPC Endpoint** để các private subnet có thể truy cập S3 mà không cần NAT Gateway.  
- Gắn **Lambda ETL** vào VPC và đảm bảo nó truy cập được cả S3 và Data Warehouse qua kết nối riêng tư.  
- Cài đặt logic ETL để đọc file JSON clickstream thô, transform và nạp chúng vào **PostgreSQL Data Warehouse** trong private subnet.  
- Kiểm chứng đầy đủ đường đi dữ liệu bằng cách xem log và chạy truy vấn SQL.

Trong phần tiếp theo (5.5), bạn sẽ sử dụng **các dashboard R Shiny** chạy trên EC2 Data Warehouse để trực quan hoá dữ liệu đã xử lý và xây dựng các view phân tích tương tác.

---

**Hình 5-11: Lớp phân tích riêng tư với Lambda ETL, Data Warehouse và S3 Gateway Endpoint**

Sơ đồ cho thấy pipeline ETL theo lô chạy hoàn toàn bên trong Analytics private subnet:

- **EventBridge** (góc dưới bên phải) trigger **Lambda ETL** theo lịch.  
- Lambda ETL đọc các file clickstream thô từ **S3 Raw Clickstream bucket** thông qua **S3 Gateway VPC Endpoint** (không cần Internet/NAT).  
- Lambda ETL ghi dữ liệu đã transform vào **EC2 PostgreSQL Data Warehouse**, và **R Shiny Server** chạy trên cùng instance đọc dữ liệu này để hiển thị các dashboard phân tích.

![Figure 5-11: Private analytics layer with ETL Lambda, Data Warehouse, and S3 Gateway Endpoint](/images/5-4-etl-s3-endpoint.png)
