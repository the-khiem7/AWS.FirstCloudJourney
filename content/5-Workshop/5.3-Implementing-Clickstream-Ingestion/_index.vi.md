---
title: "Implementing Clickstream Ingestion"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Phần này mô tả chi tiết cách các sự kiện clickstream được tạo ra trong trình duyệt, gửi tới **Amazon API Gateway**, được xử lý bởi **Lambda Ingest**, và cuối cùng được lưu dưới dạng các file JSON thô trong **S3 Raw Clickstream bucket**.

Đường ingest là **điểm vào của toàn bộ nền tảng phân tích**: nếu sự kiện không được thu thập hoặc lưu trữ đúng ở bước này, thì các bước ETL phía sau và các dashboard sẽ không còn đáng tin cậy.

---

### Sinh sự kiện ở frontend (Frontend event generation)

Frontend của site thương mại điện tử được xây bằng **Next.js** và host trên **AWS Amplify**. Một module tracking phía client chịu trách nhiệm thu thập hành vi của người dùng và gửi chúng dưới dạng các sự kiện clickstream.

Các hành vi điển hình nên được ghi lại bao gồm:

- **Page view**: trang landing, trang danh mục, trang kết quả tìm kiếm, trang chi tiết sản phẩm.  
- **Tương tác với sản phẩm**: click vào thẻ sản phẩm, xem chi tiết sản phẩm, thêm/bớt sản phẩm vào/khỏi giỏ hàng.  
- **Các bước checkout**: bắt đầu checkout, nhập thông tin giao hàng, đặt hàng.  
- **Thông tin session và danh tính**: user ID (nếu đã đăng nhập), session ID, login state, client ID.

Mã tracking ở phía frontend thường thực hiện các bước sau:

1. Lắng nghe các sự kiện trong UI (ví dụ: page load, button click).  
2. Tạo một payload JSON với tối thiểu các trường:

   - `event_id` – định danh duy nhất cho event.  
   - `event_name` – ví dụ: `"page_view"`, `"product_view"`, `"add_to_cart"`.  
   - `event_timestamp` – timestamp phía client (ISO 8601 hoặc epoch).  
   - `user_id` / `identity_source` – nếu người dùng đã đăng nhập (ví dụ: Cognito user sub).  
   - `session_id` / `client_id` – định danh session hoặc trình duyệt.  
   - `product_id`, `product_name`, `product_category`, v.v. cho các event liên quan tới sản phẩm.  
   - `page_url`, `referrer`, các trường `utm_*` để gắn nguồn traffic (tuỳ chọn).

3. Gửi payload JSON qua HTTPS tới endpoint của API Gateway (xem mục 5.3.2).

Ví dụ một payload JSON tối thiểu được gửi từ trình duyệt:

```json
{
  "event_name": "product_view",
  "event_timestamp": "2025-12-04T15:23:45.123Z",
  "user_id": "user_123",
  "login_state": "logged_in",
  "session_id": "session_abc",
  "client_id": "browser_xyz",
  "product_id": "SKU-12345",
  "product_name": "Gaming Laptop 15 inch",
  "product_category": "Laptops",
  "product_price": 1299.0,
  "page_url": "/products/sku-12345"
}
```

Payload có thể được mở rộng dần khi yêu cầu phân tích phát triển.

---

### Luồng API Gateway và Lambda Ingest

#### API Gateway HTTP API

Endpoint ingest được expose bằng **Amazon API Gateway (HTTP API)** với route ví dụ:

- Method: `POST`  
- Path: `/clickstream`  

Các đặc điểm chính:

- Endpoint chấp nhận payload JSON từ trình duyệt.  
- CORS được cấu hình để cho phép domain frontend (CloudFront / Amplify) gọi đến.  
- Route được tích hợp với **Lambda function** (Lambda proxy integration).

**Hình 5-5: API Gateway route cho POST /clickstream**

Ảnh chụp màn hình cho thấy cấu hình HTTP API cho endpoint ingest clickstream.  
Resource `/clickstream` expose một route `POST` duy nhất, được tích hợp với Lambda `clickstream-ingest`.  
Trong workshop này không cấu hình authorizer để nội dung tập trung vào ingest dữ liệu thay vì xác thực.

![Hình 5-5: API Gateway route cho POST /clickstream](/images/5-3-apigw-clickstream-route.png)


#### Lambda Ingest function

**Lambda Ingest** chịu trách nhiệm cho các việc sau:

1. **Parse request**  
   - Đọc body từ event mà API Gateway gửi vào.  
   - Kiểm tra body có chứa một JSON object hoặc mảng các object hợp lệ hay không.

2. **Validate & enrich cơ bản**  
   - Đảm bảo các trường bắt buộc (ví dụ: `event_name`, `event_timestamp`) tồn tại.  
   - Nếu timestamp phía client bị thiếu, gắn thêm timestamp phía server.  
   - Thêm các metadata như:
     - API Gateway request ID.  
     - Source IP hoặc user agent (nếu cần).  

3. **Batch và ghi vào S3**  
   - Tạo key trong Raw Clickstream bucket theo pattern phân vùng theo thời gian, ví dụ:

     ```text
     s3://<raw-bucket>/events/YYYY/MM/DD/HH/events-<uuid>.json
     ```

   - Ghi các event nhận được dưới dạng JSON array hoặc NDJSON (newline-delimited JSON), tuỳ định dạng đã chọn.

4. **Logging và xử lý lỗi**  
   - Ghi log các lỗi validate hoặc event sai format vào **CloudWatch Logs**.  
   - Trả về status code HTTP phù hợp cho API Gateway (ví dụ: `200` khi thành công, `400` khi payload không hợp lệ).

Ví dụ S3 object key cho một batch event được ghi nhận lúc 15:00 ngày 04/12/2025:

```text
events/2025/12/04/15/events-a1b2c3d4.json
```

**Hình 5-6: Tổng quan Lambda Ingest**

Ảnh chụp màn hình hiển thị function `clickstream-lambda-ingest` trong AWS Lambda console.  
Sơ đồ minh hoạ việc function được trigger bởi HTTP API Gateway và sử dụng cấu hình nhẹ (128 MB memory, timeout ngắn) – đủ để validate payload JSON, enrich metadata và ghi các batch event vào S3 Raw Clickstream bucket.

![Hình 5-6: Lambda Ingest function overview](/images/5-3-lambda-ingest-config.png)

---

### Kiểm thử thủ công từ frontend và Postman

Để xác nhận pipeline ingest hoạt động như mong đợi, ta thực hiện hai kiểu kiểm thử bổ sung.

#### Kiểm thử end-to-end từ frontend thật

1. Mở domain CloudFront của ứng dụng thương mại điện tử, ví dụ:

   ```text
   https://dxxxxxxxx.cloudfront.net
   ```

2. Đăng nhập qua **Amazon Cognito** bằng một tài khoản test.  
3. Thực hiện một chuỗi hành động, chẳng hạn:

   - Mở trang chủ và một trang danh mục.  
   - Xem chi tiết hai hoặc ba sản phẩm.  
   - Thêm ít nhất một sản phẩm vào giỏ hàng rồi xoá ra.  
   - Bắt đầu flow checkout và đi tới bước xác nhận cuối cùng (việc đặt đơn hàng thật là tuỳ chọn).

4. Dùng Developer Tools của trình duyệt (tab Network) để kiểm tra:

   - Các request đang được gửi tới `POST /clickstream`.  
   - Request body chứa các trường JSON mong đợi như `event_name`, `product_id`, `session_id`, v.v.  
   - Status code trả về từ API Gateway là `200`.

#### Kiểm thử trực tiếp bằng Postman hoặc Thunder Client

Để kiểm thử có kiểm soát, bạn cũng có thể gửi các event synthetic bằng API client:

- Method: `POST`  
- URL:  

  ```text
  https://<api-id>.execute-api.<region>.amazonaws.com/clickstream
  ```

- Header:

  ```text
  Content-Type: application/json
  ```

- Body:

  ```json
  {
    "event_name": "page_view",
    "event_timestamp": "2025-12-04T16:00:00.000Z",
    "user_id": "test_user_manual",
    "login_state": "anonymous",
    "session_id": "session_manual_001",
    "client_id": "postman_client",
    "page_url": "/manual-test"
  }
  ```

Cách làm này hữu ích để kiểm thử xử lý lỗi, các rule validate, hoặc các edge case mà không phụ thuộc vào code frontend.

---

### Kiểm tra dữ liệu trong S3 Raw Clickstream bucket

Sau khi đã gửi event, bước tiếp theo là xác minh chúng đã được lưu thành công vào S3.

1. Mở **Amazon S3 Console** và chọn Raw Clickstream bucket, ví dụ:

   ```text
   clickstream-raw-<account>-<region>
   ```

2. Điều hướng theo cấu trúc prefix:

   ```text
   events/YYYY/MM/DD/HH/
   ```

   Ví dụ:

   ```text
   events/2025/12/04/15/
   ```

3. Xác nhận rằng đã có một hoặc nhiều object với tên tương tự:

   ```text
   events-a1b2c3d4.json
   events-f9e8d7c6.json
   ```

   được tạo ra.

4. Tải một trong các file này về và mở bằng text editor (VS Code, Notepad++, v.v.):

   - Kiểm tra JSON có hợp lệ hay không.  
   - Kiểm tra các trường như `event_name`, `event_timestamp`, `user_id`, `session_id`, `product_id`, v.v. có khớp với các thao tác bạn đã thực hiện trên website hay không.  
   - Đảm bảo nhiều event được nhóm lại hợp lý (ví dụ: một mảng các JSON object).

5. Ghi chú lại:

   - Kích thước file xấp xỉ (KB/MB).  
   - Số lượng event trong mỗi file.  
   - Tần suất xuất hiện file mới (tuỳ thuộc vào cách Lambda Ingest batch và ghi).

Những quan sát này sẽ hữu ích khi tinh chỉnh kích thước batch ETL và lịch chạy ở các phần sau.

**Hình 5-7: Các object JSON clickstream thô trong S3 bucket**

Ảnh chụp màn hình hiển thị Raw Clickstream bucket tại prefix sâu nhất theo giờ.  
Nhiều object JSON được tạo bởi Lambda Ingest, mỗi object đại diện cho một batch các sự kiện clickstream trong giờ đó.  
Tên file sử dụng pattern dựa trên UUID (ví dụ: `event-a8bdf0c0-...json`), và cột `Size` cho biết nhanh số lượng event tương đối trong mỗi file.

![Hình 5-7: Raw clickstream JSON objects in the S3 bucket](/images/5-3-s3-raw-objects.png)

---

### Tổng kết và mối liên hệ với các thành phần downstream

Kết thúc phần này, bạn nên đã có:

- Một đường ingest hoạt động từ **browser → API Gateway → Lambda Ingest → S3 Raw bucket**.  
- Xác minh rằng event có cấu trúc đúng và chứa đủ metadata cần thiết.  
- Xác nhận Raw Clickstream bucket đang sử dụng **layout phân vùng theo thời gian**, phù hợp cho xử lý theo lô.

Trong phần tiếp theo (5.4), trọng tâm sẽ chuyển sang **lớp phân tích riêng tư (private analytics layer)**, nơi một **ETL Lambda chạy trong VPC** đọc các file JSON thô này qua **S3 Gateway VPC Endpoint**, transform chúng và nạp vào **PostgreSQL Data Warehouse**.

---

**Hình 5-8: Luồng ingest từ trình duyệt tới S3 Raw bucket**

Sơ đồ minh hoạ chuỗi:

- Trình duyệt (user actions) → **CloudFront** → **Amplify** (ứng dụng Next.js).  
- Tracking ở frontend gửi các request HTTP tới **Amazon API Gateway (HTTP API)** trên route `POST /clickstream`.  
- API Gateway gọi **Lambda Ingest**, function này validate và enrich event.  
- Lambda Ingest ghi **các object JSON dạng batch** vào **S3 Raw Clickstream bucket** theo key phân vùng thời gian, chẳng hạn `events/YYYY/MM/DD/HH/events-<uuid>.json`.

![Hình 5-8: Ingestion flow from frontend to S3 Raw bucket](/images/5-3-ingestion-flow.png)
