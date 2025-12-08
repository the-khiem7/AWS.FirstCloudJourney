---
title: "Visualizing Analytics with Shiny Dashboards"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Trong các phần trước, bạn đã ingest các sự kiện clickstream vào **S3 Raw Clickstream bucket** và sử dụng một **Lambda ETL có gắn VPC (VPC-enabled ETL Lambda)** để nạp dữ liệu đã được xử lý vào **PostgreSQL Data Warehouse** trong private subnet.

Phần này tập trung vào “**chặng cuối**” của pipeline:

- Xác nhận rằng job ETL đã nạp đủ và đúng dữ liệu vào Data Warehouse.  
- Sử dụng **R Shiny dashboards** chạy trên cùng EC2 với Data Warehouse để khám phá hành vi người dùng, các phễu (funnel) và hiệu suất sản phẩm.

Mục tiêu không chỉ là “xem vài biểu đồ”, mà là **kết nối từng trực quan hoá với mô hình dữ liệu bên dưới và các câu hỏi nghiệp vụ**.

---

### Trigger batch ETL và kiểm tra dữ liệu trong Data Warehouse

Trước khi mở dashboard, hãy đảm bảo Data Warehouse đã được cập nhật dữ liệu mới.

#### Bước 1 – Sinh các sự kiện clickstream mới

1. Mở domain CloudFront của ứng dụng thương mại điện tử, ví dụ:

   ```text
   https://dxxxxxxxx.cloudfront.net
   ```

2. Đăng nhập bằng một user test qua **Amazon Cognito**.  
3. Thực hiện một phiên duyệt web “giống thật”, chẳng hạn:

   - Vào trang chủ và ít nhất một trang danh mục.  
   - Tìm kiếm sản phẩm hoặc lọc theo brand/category.  
   - Mở chi tiết từ ba sản phẩm trở lên.  
   - Thêm một–hai sản phẩm vào giỏ hàng.  
   - Xoá một sản phẩm, thay đổi số lượng, và đi tiếp vào flow checkout.  
   - Tuỳ chọn: hoàn thành đặt hàng để tạo các sự kiện purchase.

4. Chờ khoảng 1–2 phút để đảm bảo tất cả event đã được gửi tới **API Gateway → Lambda Ingest → S3 Raw Clickstream bucket**.

#### Bước 2 – Chạy ETL job

Có ba cách chính để chạy Lambda ETL:

- **A. Chờ EventBridge chạy theo lịch**  
  - Nếu rule ETL đã được cấu hình chạy mỗi 15 hoặc 30 phút, bạn có thể đơn giản là chờ lần trigger tiếp theo.

- **B. Trigger thủ công qua EventBridge**  
  1. Mở console **Amazon EventBridge**.  
  2. Vào **Rules** và chọn rule ETL, ví dụ:

     ```text
     clickstream-etl-schedule
     ```

  3. Chọn **Actions → Run now** để chạy rule ngay lập tức.

- **C. Trigger thủ công qua Lambda console**  
  1. Mở **Lambda Console** và chọn function ETL, ví dụ:

     ```text
     clickstream-etl-lambda
     ```

  2. Ở tab **Test**, tạo hoặc dùng lại một test event (payload `{}` là đủ nếu code bỏ qua input).  
  3. Bấm **Test** để invoke Lambda ETL.

#### Bước 3 – Kiểm tra ETL trong CloudWatch Logs

1. Mở **CloudWatch Logs** trong AWS console.  
2. Điều hướng tới log group của Lambda ETL.  
3. Mở log stream mới nhất và tìm các dòng như:

   - “Listing S3 objects under prefix `events/YYYY/MM/DD/HH/` …”  
   - “Read N files, parsed M events.”  
   - “Inserted K rows into `fact_events`, L rows into `dim_products`, …”  
   - Bất kỳ error hoặc stack trace nào (nếu có).

Nếu gặp lỗi, hãy kiểm tra:

- Quyền IAM cho S3 và database.  
- Cấu hình VPC (subnet, route table, Gateway Endpoint).  
- Kết nối tới database (host, port, thông tin đăng nhập).

**Hình 5-12: CloudWatch logs cho lần chạy ETL gần nhất**

Ảnh chụp màn hình hiển thị log stream trong CloudWatch cho một lần chạy gần đây của Lambda ETL.  
Các entry `INIT_START`, `START`, `END` và `REPORT` xác nhận function chạy thành công, thời lượng và memory usage nằm trong phạm vi mong đợi.  
Việc kiểm tra log giúp đảm bảo Data Warehouse đã có dữ liệu mới trước khi mở Shiny dashboard.

![Hình 5-12: CloudWatch logs cho lần chạy ETL gần nhất](/images/5-5-etl-cloudwatch-logs.png)

#### Bước 4 – Kiểm tra dữ liệu bằng truy vấn SQL

Sau khi ETL chạy thành công, hãy xác minh Data Warehouse đã được cập nhật.

1. Kết nối vào **EC2 Data Warehouse** bằng **AWS Systems Manager Session Manager** (không cần SSH key hay bastion host).  
2. Từ shell của Session Manager, dùng `psql` hoặc một client SQL bất kỳ để kết nối tới PostgreSQL DW.

Chạy một số truy vấn cơ bản:

```sql
-- 1. Tổng số event trong bảng fact
SELECT COUNT(*) AS total_events
FROM fact_events;
```

```sql
-- 2. Số event theo loại (page view, product view, add-to-cart, ...)
SELECT event_name, COUNT(*) AS total_events
FROM fact_events
GROUP BY event_name
ORDER BY total_events DESC;
```

```sql
-- 3. Top 10 sản phẩm được xem nhiều nhất
SELECT
    product_id,
    product_name,
    COUNT(*) AS view_count
FROM fact_events
WHERE event_name = 'product_view'
GROUP BY product_id, product_name
ORDER BY view_count DESC
LIMIT 10;
```

Kiểm tra sâu hơn (tuỳ chọn):

```sql
-- 4. Dạng funnel đơn giản: từ product view đến add-to-cart
SELECT
    product_id,
    SUM(CASE WHEN event_name = 'product_view'  THEN 1 ELSE 0 END) AS product_views,
    SUM(CASE WHEN event_name = 'add_to_cart'   THEN 1 ELSE 0 END) AS add_to_cart_events
FROM fact_events
GROUP BY product_id
ORDER BY product_views DESC
LIMIT 10;
```

So sánh kết quả với phiên test:

- Bạn đã xem ít nhất chừng đó sản phẩm chưa?  
- Các sản phẩm bạn tương tác có nằm trong top danh sách không?  
- Nếu bạn hoàn tất một đơn hàng, bạn có nhìn thấy các event liên quan trong dữ liệu không?

---

### Truy cập Shiny dashboards (từ EC2 private)

**R Shiny Server** chạy trên instance EC2 private chung với Data Warehouse và **không có public IP**. Để truy cập an toàn, sử dụng **AWS Systems Manager Session Manager port forwarding**—không cần SSH key hay bastion host.

#### Truy cập Shiny qua Session Manager Port Forwarding

1. Đảm bảo **Session Manager plugin** cho AWS CLI đã được cài đặt trên máy local của bạn.  
   - Hướng dẫn cài đặt: [AWS Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

2. Chạy lệnh sau để forward cổng Shiny (mặc định 3838) về máy local:

   ```bash
   aws ssm start-session \
       --target <instance-id> \
       --document-name AWS-StartPortForwardingSession \
       --parameters '{"portNumber":["3838"],"localPortNumber":["8080"]}'
   ```

   Thay `<instance-id>` bằng EC2 instance ID của Data Warehouse instance.

3. Giữ terminal session này mở. Bạn sẽ thấy thông báo kiểu:

   ```text
   Starting session with SessionId: ...
   Port 8080 opened for sessionId ...
   ```

4. Mở trình duyệt trên máy local và truy cập:

   ```text
   http://localhost:8080/
   ```

   hoặc app Shiny cụ thể, ví dụ:

   ```text
   http://localhost:8080/clickstream-analytics
   ```

#### Lợi ích của Session Manager Port Forwarding

- **Không phơi bày SSH**: EC2 private không cần inbound SSH rule hay public IP.  
- **Không cần bastion host**: Loại bỏ nhu cầu quản lý và bảo mật jump server.  
- **Kiểm soát truy cập qua IAM**: Quyền được quản lý bằng IAM policy, với đầy đủ audit trail trong CloudTrail.  
- **Tunnel mã hóa**: Toàn bộ traffic được mã hóa qua HTTPS, luôn ở trong mạng AWS thông qua VPC Interface Endpoints.

---

### Khám phá các dashboard

Sau khi truy cập được trang chủ Shiny hoặc app cụ thể, bạn sẽ thấy một hoặc nhiều dashboard được xây trên Data Warehouse.

Một số view thường gặp:

#### Dashboard Phễu / Hành trình người dùng (Funnel / User Journey)

Hiển thị cách người dùng đi qua các bước chính, ví dụ:

1. `page_view` → 2. `product_view` → 3. `add_to_cart` → 4. `checkout_start` → 5. `purchase`

Các trực quan hoá phổ biến:

- Biểu đồ funnel với số lượng ở từng bước.  
- Tỷ lệ rơi rụng (drop-off) giữa các bước liên tiếp.  
- Bộ lọc theo khoảng thời gian, loại thiết bị, nguồn traffic.

Câu hỏi bạn có thể trả lời:

- Có bao nhiêu người dùng bắt đầu từ product view và đi tới bước checkout?  
- Người dùng rơi rụng chủ yếu ở đâu (trước add-to-cart, hay trong quá trình checkout)?  
- Phễu có thay đổi theo thời gian hoặc theo từng nguồn traffic không?

#### Dashboard Hiệu suất sản phẩm (Product Performance)

Tập trung vào các chỉ số ở cấp sản phẩm:

- Sản phẩm được xem nhiều nhất (`product_view`).  
- Sản phẩm có nhiều event add-to-cart hoặc purchase.  
- Tỷ lệ chuyển đổi theo sản phẩm (add-to-cart / views, purchase / views).

Trực quan hoá thường gặp:

- Biểu đồ cột xếp hạng sản phẩm theo view hoặc purchase.  
- Bảng hiển thị tên sản phẩm, category và các chỉ số tương tác chính.  
- Bộ lọc theo category, brand, khoảng giá…

Câu hỏi bạn có thể trả lời:

- Sản phẩm nào thu hút nhiều lượt xem nhưng chuyển đổi thấp?  
- Category hoặc brand nào hoạt động tốt nhất?  
- Có sản phẩm nào gần như không được xem và có thể cần được quảng bá thêm?

#### Dashboard Chuỗi thời gian / Mức độ hoạt động (Time Series / Activity Over Time)

Theo dõi cách hoạt động của người dùng thay đổi theo thời gian:

- Số event mỗi giờ hoặc mỗi ngày.  
- Các đường riêng cho page view, product view, add-to-cart, purchase.  
- Tuỳ chọn breakdown theo loại thiết bị hoặc nguồn traffic.

Câu hỏi bạn có thể trả lời:

- Thời điểm nào trong ngày có hoạt động duyệt web cao nhất?  
- Có ngày nào trong tuần có tỷ lệ chuyển đổi tốt hơn không?  
- Các chiến dịch hoặc chương trình khuyến mãi (nếu được ghi nhận) có trùng với các đỉnh (spike) hoạt động không?

---

### Liên hệ giữa dashboard và schema Data Warehouse

Mỗi dashboard được vận hành bởi các truy vấn SQL chạy trên các bảng trong Data Warehouse, ví dụ:

- `fact_events` – dữ liệu event chi tiết.  
- `dim_products` – thuộc tính sản phẩm (tên, category, brand, v.v.).  
- `dim_users` – thuộc tính người dùng (đã đăng ký vs khách, segment, v.v.).  
- `fact_sessions` hoặc `fact_funnels` – các bảng fact đã tổng hợp theo session/funnel (nếu có).

Khi bạn tương tác với các filter trong Shiny (chọn khoảng thời gian, category, loại event…), app Shiny thường:

1. Xây dựng truy vấn SQL dựa trên tham số bạn chọn.  
2. Gửi truy vấn tới PostgreSQL.  
3. Nhận về kết quả đã tổng hợp.  
4. Render thành biểu đồ hoặc bảng trong trình duyệt.

Bài tập gợi ý:

- Mở mã nguồn app Shiny (các script R) trên EC2.  
- Tìm các truy vấn SQL được dùng cho từng widget.  
- So sánh những truy vấn đó với các truy vấn **SQL thủ công** mà bạn đã chạy ở trên.

Điều này giúp bạn tự tin rằng:

- Dashboard nhất quán với dữ liệu bên dưới.  
- Bạn có thể tái tạo các con số chính bằng SQL nếu cần.

---

### Tổng kết

Hoàn thành phần này, bạn đã:

- Đảm bảo **batch ETL** đã nạp dữ liệu clickstream mới vào PostgreSQL Data Warehouse.  
- Xác nhận dữ liệu bằng cách chạy **các truy vấn SQL** trực tiếp (tổng số event, lượt xem sản phẩm, các chỉ số funnel cơ bản).  
- Truy cập **R Shiny dashboards** chạy trên EC2 private thông qua port forwarding an toàn.  
- Khám phá các dashboard về hành trình người dùng, hiệu suất sản phẩm và hoạt động theo thời gian.  
- Kết nối các trực quan Shiny với schema và truy vấn trong Data Warehouse.

Trong phần tiếp theo (**5.6 Summary & Clean up**), bạn sẽ tóm tắt lại các điểm chính từ workshop và rà soát những tài nguyên AWS cần dừng hoặc xoá để tránh phát sinh chi phí không cần thiết.

---

**Hình 5-13: Shiny dashboard cho phân tích clickstream**

Ảnh chụp màn hình Shiny dashboard được xây trên PostgreSQL Data Warehouse:

- Phễu hành trình người dùng từ product view đến purchase.  
- Biểu đồ chuỗi thời gian số event mỗi ngày (page view, product view, add-to-cart, purchase).  
- Bảng top sản phẩm hiển thị các item có tương tác và chuyển đổi cao nhất.  
- Bộ lọc ở phía trên (khoảng ngày, loại event, thiết bị, nguồn traffic) để phân tích theo nhiều lát cắt khác nhau.

![Hình 5-13: Shiny dashboard cho phân tích clickstream](/images/5-5-shiny-dashboard.png)
