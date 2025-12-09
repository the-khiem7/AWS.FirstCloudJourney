---
title: "Triển khai thu nhận Clickstream"
weight: 53
chapter: false
pre: " <b> 5.3. </b> "
---

## 5.3.1 Tổng quan luồng thu nhận

Luồng dữ liệu mức cao:

1. Người dùng tương tác với frontend Next.js (`ClickSteam.NextJS`) được host trên Amplify.  
2. JavaScript phía frontend đóng gói metadata clickstream (page/user/session/product) thành payload JSON.  
3. Trình duyệt gửi yêu cầu `POST` tới `clickstream-http-api` tại `POST /clickstream`.  
4. API Gateway chuyển tiếp yêu cầu tới `clickstream-lambda-ingest`.  
5. Lambda Ingest bổ sung metadata `_ingest` và ghi mỗi sự kiện thành một tệp JSON vào:
   - `s3://clickstream-s3-ingest/events/YYYY/MM/DD/HH/event-<uuid>.json` (phân vùng theo giờ UTC)  

Thiết kế giữ **stateless** và **append-only**, phù hợp cho ETL lô phía sau và chèn idempotent.

---

## 5.3.2 Thiết kế S3 Bucket

Hai bucket liên quan:

1. **Bucket cho các Asset** - `clickstream-s3-sbw`  
   - Lưu trữ tài nguyên website: ảnh sản phẩm, tệp tĩnh.  
   - Không dùng cho sự kiện clickstream.

2. **Bucket Clickstream RAW** - `clickstream-s3-ingest`  
   - Chỉ lưu sự kiện clickstream JSON thô.  
   - Phân vùng theo giờ UTC: `events/YYYY/MM/DD/HH/`  
   - Đặt tên tệp: `event-<uuid>.json`  

Phân vùng theo giờ giúp ETL lô dễ hơn (vd xử lý giờ trước hoặc prefix ngày/giờ cụ thể).

![Sự kiện bucket S3 RAW](/images/aws-s3-clickstream-ingest-events.png)

---

## 5.3.3 Thiết kế Lambda Ingest - `clickstream-lambda-ingest`

![Lambda Ingest](/images/aws-lambda-clickstream-ingest-config.png)

### Nhiệm vụ

Hàm `clickstream-lambda-ingest`:

- Phân tích payload JSON nhận từ API Gateway.  
- Chỉ thêm metadata `_ingest`: `receivedAt`, `sourceIp`, `userAgent`, `method`, `path`, `requestId`, `apiId`, `stage`, `traceId`.  
- Ghi phần thân sự kiện **đúng như client gửi** (không tự điền user/session/product phía server) vào S3.

### Quyền IAM

Vai trò thực thi cần:

- `s3:PutObject` trên: `arn:aws:s3:::clickstream-s3-ingest/events/*`  
- Các API CloudWatch Logs để ghi log.

Hàm không cần quyền đọc.

---

## 5.3.4 API Gateway HTTP API - `clickstream-http-api`

HTTP API cung cấp endpoint HTTPS công khai cho ingestion:

- Route:
  - `POST /clickstream` -> Lambda `clickstream-lambda-ingest`  

![Route POST /clickstream](/images/aws-apigw-clickstream-routes.png)

Tùy chọn khuyến nghị:

- Bật **CORS** để frontend Amplify có thể gọi từ domain của nó.  
- Bật **access log** tới CloudWatch log group để debug.  
- (Tùy chọn) Gắn **API key** hoặc **Cognito authorizer** nếu muốn hạn chế ingestion.

---

## 5.3.5 Frontend Clickstream Publisher (Logic)

### Danh tính & tính idempotent
- Sinh `eventId` cho mỗi sự kiện (UUID) để đảm bảo nguyên lý idempotent.
- Giữ `clientId` trong `localStorage` (cố định theo trình duyệt).
- Giữ `sessionId` trong `sessionStorage` (timeout nhàn rỗi 30 phút) và `isFirstVisit`.

### Metadata người dùng, trang và click
- Người dùng/xác thực (nếu có): `userId`, `userLoginState`, tùy chọn `identity_source`.
- Trang/click: `pageUrl`, `referrer`, metadata phần tử cho click (tag/id/role/text/dataset).

### Ngữ cảnh sản phẩm
- Gửi dưới dạng `product.{id,name,category,brand,price,discountPrice,urlPath}`; ETL ánh xạ tới các cột DW `context_product_*`.

### Phạm vi sự kiện
- Tự động: `page_view`, `click` toàn cục.
- Tùy chỉnh/sản phẩm: `home_view`, `category_view`, `product_view`, `add_to_cart_click`, `remove_from_cart_click`, `wishlist_toggle`, `share_click`, `login_open`, `login_success`, `logout`, `checkout_start`, `checkout_complete`.

### Mapping sự kiện domain 
- `home_view`: load trang chủ (component tracker).
- `category_view`: render danh sách danh mục (slug/params).
- `product_view`: render chi tiết sản phẩm (có ngữ cảnh sản phẩm).
- `add_to_cart_click` / `remove_from_cart_click`: handler thêm/xóa giỏ.
- `wishlist_toggle`: handler nút wishlist.
- `share_click`: handler nút chia sẻ.
- `login_open` / `login_success` / `logout`: luồng auth.
- `checkout_start` / `checkout_complete`: bước vào checkout và hoàn tất.

### Component & wiring
- `lib/clickstreamClient.ts`: xử lý danh tính/phiên, builder cơ sở, log console, `fetch` fire-and-forget tới `NEXT_PUBLIC_CLICKSTREAM_ENDPOINT` (biến môi trường bắt buộc).
- `lib/clickstreamEvents.ts`: helper domain bọc `trackCustom` và dựng ngữ cảnh sản phẩm/giỏ/đơn hàng.
- `contexts/ClickstreamProvider.tsx` + `app/layout.tsx`: nối provider toàn cục, tự động `page_view`, listener click toàn cục.
- Component tracker: `HomeTracker.tsx`, `CategoryTracker.tsx`, `ProductViewTracker.tsx` bỏ qua auto page_view và phát sự kiện domain.
- UI đã gắn: `AddToCartButton.tsx`, `FavoriteButton.tsx`, `app/(client)/cart/page.tsx` phát sự kiện thêm/xóa giỏ, wishlist, checkout và đánh dấu nút với `global-clickstream-ignore-click` để tránh click toàn cục trùng lặp.

### Hành vi runtime
- Chỉ chạy phía client (không có hiệu ứng SSR).
- Log mọi sự kiện lên console; nếu thiếu endpoint, chạy dry-run và cảnh báo một lần.
- Lỗi mạng không chặn UI.

## 5.3.6 Chiếu trường (frontend -> S3 -> DW)

| Trường / Khối          | Payload frontend                              | Raw S3 (sau Ingest)                                             | DW (PostgreSQL)                               | Ghi chú                                  |
| ---                    | ---                                           | ---                                                             | ---                                           | ---                                      |
| event_id               | `eventId` sinh ở client                       | giữ nguyên payload                                              | `event_id` (ánh xạ từ eventId; ETL fallback UUID) | Khóa chính, ON CONFLICT DO NOTHING      |
| event_timestamp        | -                                             | - (có LastModified của S3)                                      | `_ingest.receivedAt` > payload > LastModified | ETL suy ra                               |
| event_name             | `eventName`                                   | `eventName`                                                     | `event_name`                                  |                                           |
| page_url               | `pageUrl`                                     | `pageUrl`                                                       | -                                             | Không lưu trong DW                       |
| referrer               | `referrer`                                    | `referrer`                                                      | -                                             | Không lưu trong DW                       |
| user_id                | `userId`                                      | `userId`                                                        | `user_id`                                     |                                           |
| user_login_state       | `userLoginState`                              | `userLoginState`                                                | `user_login_state`                            |                                           |
| identity_source        | tùy chọn                                      | tùy chọn                                                        | `identity_source`                             | Cần frontend/auth cung cấp               |
| client_id              | `clientId`                                    | `clientId`                                                      | `client_id`                                   |                                           |
| session_id             | `sessionId`                                   | `sessionId`                                                     | `session_id`                                  |                                           |
| is_first_visit         | `isFirstVisit`                                | `isFirstVisit`                                                  | `is_first_visit`                              |                                           |
| product id             | `product.id`                                  | `product.id`                                                    | `context_product_id`                          |                                           |
| product name           | `product.name`                                | `product.name`                                                  | `context_product_name`                        |                                           |
| product category       | `product.category`                            | `product.category`                                              | `context_product_category`                    |                                           |
| product brand          | `product.brandName` / `brand?.name` / `brand?.title` / `brand` | `product.brand` hoặc `product.brandName`                        | `context_product_brand`                       | Frontend ánh xạ brand từ các trường DB  |
| product price          | `product.price`                               | `product.price`                                                 | `context_product_price`                       | BIGINT                                   |
| product discount price | `product.discountPrice`                       | `product.discountPrice`                                         | `context_product_discount_price`              | BIGINT                                   |
| product url path       | `product.urlPath`                             | `product.urlPath`                                               | `context_product_url_path`                    |                                           |
| element metadata       | `element.{tag,id,role,text,dataset}`          | `element...`                                                    | -                                             | Không lưu (cần thay đổi schema)         |
| ingest metadata        | -                                             | `_ingest.{receivedAt,sourceIp,userAgent,method,path,requestId,apiId,stage,traceId}` | -                                             | Không lưu; chỉ có khi ETL               |

Mẫu gọi (mỗi yêu cầu một sự kiện):

```ts
await fetch("https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/clickstream", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(eventPayload),
  keepalive: true,
});
```

ETL ánh xạ `eventId -> event_id` (PK với ON CONFLICT DO NOTHING), suy ra `event_timestamp` từ `_ingest.receivedAt` > payload > S3 LastModified, và chèn vào `clickstream_dw.public.clickstream_events`.

---

## 5.3.6 Kiểm thử & xác nhận

Để kiểm tra thu nhận:

1. Dùng UI của app Amplify:
   - Duyệt vài trang sản phẩm  
   - Thêm sản phẩm vào giỏ  
2. Kiểm tra bucket S3 `clickstream-s3-ingest`:
   - Điều hướng tới `events/YYYY/MM/DD/HH/`  
   - Xác nhận xuất hiện các tệp mới `event-<uuid>.json`.  
3. Mở một tệp JSON:
   - Kiểm tra metadata `_ingest` và ngữ cảnh sản phẩm có mặt.  
4. Xem log:
   - API Gateway access logs  
   - Lambda function logs  
