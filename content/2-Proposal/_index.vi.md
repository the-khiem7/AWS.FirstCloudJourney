---
title: "Bản đề xuất"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Team Super Beast Warrior(SBW)

# Batch-based Clickstream Analytics Platform

### 1. Tóm tắt

Dự án này nhằm **thiết kế và triển khai Batch-based Clickstream Analytics Platform** cho một website thương mại điện tử chuyên về **máy tính và phụ kiện** (giao diện frontend của website được tích hợp một **JavaScript SDK** nhẹ để gửi dữ liệu hoạt động của người dùng như **clicks**, **views**, **searches** tới **backend API**) bằng cách sử dụng **AWS Cloud Services**.
Hệ thống thu thập dữ liệu tương tác của người dùng (như **clicks**, **searches**, và **page visits**) từ website và lưu trữ chúng trong **Amazon S3** dưới dạng **raw logs**. Cứ mỗi giờ, **Amazon EventBridge** sẽ kích hoạt **AWS Lambda** để xử lý và chuyển đổi dữ liệu trước khi nạp vào **data warehouse** được lưu trữ trên **Amazon EC2**.

Dữ liệu đã được xử lý sẽ được visualize thông qua **R Shiny dashboards**, giúp chủ cửa hàng có thể theo dõi business insights như hành vi khách hàng, mức độ phổ biến của sản phẩm, và xu hướng tương tác trên website.

Kiến trúc này tập trung vào **batch analytics**, **ETL pipeline**, và **business intelligence**, đồng thời đảm bảo **bảo mật (security)**, **khả năng mở rộng (scalability)**, và **hiệu quả chi phí (cost efficiency)** thông qua việc tận dụng các **AWS managed services**.

### 2. Vấn đề đặt ra

#### Vấn đề hiện tại là gì?

Các website E-commerce tạo ra một lượng lớn **clickstream data** — bao gồm product views, cart actions, và search activities — chứa đựng nhiều business insights có giá trị.

Tuy nhiên, các cửa hàng **small và medium-sized** thường thiếu infrastructure và expertise cần thiết để collect, process, và analyze dữ liệu này một cách hiệu quả.

Kết quả là, họ gặp khó khăn trong việc:

- Hiểu hành vi mua hàng của khách hàng (customer purchasing behavior)
- Xác định sản phẩm hoạt động hiệu quả nhất (top-performing products)
- Tối ưu hóa marketing campaigns** và **hiệu suất website (website performance)
- Ra quyết định về tồn kho (inventory) và giá cả (pricing) dựa trên dữ liệu (data-driven decisions)

#### Giải pháp

Dự án này giới thiệu một **AWS-based batch clickstream analytics** system, tự động **collect** dữ liệu tương tác của người dùng từ website mỗi giờ, process thông qua serverless functions, và lưu trữ vào central data warehouse trên **Amazon EC2**.

Kết quả được visualize bằng **R Shiny dashboards**, giúp chủ cửa hàng có được actionable insights về hành vi khách hàng (customer behavior) và cải thiện hiệu suất kinh doanh tổng thể (overall business performance).

#### Lợi ích và hoàn vốn đầu tư

- **Data-driven decision making**: Khám phá sở thích của khách hàng, sản phẩm phổ biến và xu hướng mua sắm.
- **Scalable and modular design**: Dễ dàng mở rộng để xử lý nhiều người dùng hơn hoặc tích hợp thêm các nguồn dữ liệu mới.
- **Cost-efficient batch processing**: Giảm chi phí tính toán liên tục bằng cách vận hành theo lịch trình hàng giờ.
- **Business insight enablement**: Giúp chủ cửa hàng tối ưu hóa chiến lược bán hàng và cải thiện doanh thu dựa trên phân tích có cơ sở dữ liệu.

### 3. Kiến trúc giải pháp

![Architecture](/images/2-Proposal/AWS_Architecture_ver4.png)

### Dịch vụ AWS sử dụng

- **Amazon Cognito**: Quản lý quá trình xác thực và phân quyền người dùng cho cả quản trị viên và khách hàng của website, đảm bảo quyền truy cập an toàn vào nền tảng e-commerce.
- **Amazon S3**: Hoạt động như một lớp lưu trữ dữ liệu tập trung — lưu trữ giao diện website tĩnh (static website front-end) và các clickstream logs thô được thu thập từ tương tác người dùng. Ngoài ra, nó còn tạm thời lưu trữ các batch files trước khi được xử lý và chuyển đến data warehouse.
- **Amazon CloudFront**: Phân phối nội dung website tĩnh trên toàn cầu với độ trễ thấp, cải thiện trải nghiệm người dùng và lưu trữ cache gần khách hàng hơn.
- **Amazon API Gateway**: Đóng vai trò là điểm đầu vào chính cho các API calls từ website, cho phép gửi dữ liệu an toàn (như clickstream hoặc browsing activity) vào AWS.
- **AWS Lambda**: Thực thi các serverless functions để tiền xử lý và tổ chức clickstream data được tải lên S3. Nó cũng xử lý các tác vụ chuyển đổi dữ liệu được EventBridge kích hoạt theo lịch trước khi nạp vào data warehouse.
- **Amazon EventBridge**: Lên lịch và điều phối các batch workflows — ví dụ, kích hoạt Lambda functions mỗi giờ để xử lý và di chuyển clickstream data từ S3 vào EC2 data warehouse.
- **Amazon EC2 (Data Warehouse)**: Đóng vai trò là môi trường data warehouse, chạy PostgreSQL hoặc các cơ sở dữ liệu quan hệ khác phục vụ cho batch analytics, trend analysis và business reporting. Các instances này được triển khai trong private subnet của VPC để đảm bảo cô lập mạng và an toàn.
- **R Shiny (on EC2)**: Lưu trữ các dashboards tương tác hiển thị các insights đã được xử lý theo batch, giúp doanh nghiệp phân tích hành vi khách hàng, sản phẩm phổ biến và cơ hội bán hàng.
- **AWS IAM**: Quản lý quyền truy cập và chính sách nhằm đảm bảo chỉ những người dùng và thành phần AWS được ủy quyền mới có thể tương tác với dữ liệu và dịch vụ.
- **Amazon CloudWatch**: Thu thập và giám sát các metrics, logs, và trạng thái của các scheduled jobs từ Lambda và EC2 để duy trì độ tin cậy và khả năng quan sát hiệu suất hệ thống.
- **Amazon SNS**: Gửi thông báo hoặc cảnh báo khi batch jobs hoàn thành, thất bại hoặc gặp lỗi, đảm bảo doanh nghiệp kịp thời nắm bắt tình trạng vận hành.

### 4. Triển khai kỹ thuật

#### End-to-end data flow

1. Auth (Cognito)

- Trình duyệt xác thực với Amazon Cognito (Hosted UI hoặc JS SDK).
- ID token (JWT) được lưu trong bộ nhớ; SDK tự động gắn `Authorization: Bearer <JWT>` cho các API calls.

2. Static web (CloudFront + S3)

- SPA/assets được lưu trữ trên S3; CloudFront đứng phía trước với OAC, gzip/brotli, HTTP/2, và WAF managed rules.
- Trang web tải một analytics SDK nhỏ thu thập các events và gửi đến API Gateway (bên dưới).

3. Event ingest (API Gateway)

- `POST /v1/events` (HTTP API). CORS bị giới hạn theo site origin; JWT authorizer xác thực Cognito token (hoặc API key cho luồng anonymous).
- Các request được chuyển tiếp đến Lambda.

4. Security & Ops

- IAM được cấu hình least-privilege cho từng thành phần.
- CloudWatch ghi log, metrics và cảnh báo trên API 5xx, Lambda errors, throttles, Shiny health.
- SNS gửi thông báo khi có alarms hoặc DLQ tăng.

5. Processing & storage (Lambda → S3 batch buffer → EventBridge → Lambda (ETL) → PostgreSQL trên EC2 (data warehouse) → Shiny)

- Ingest Lambda xác thực và enrich các events, sau đó append-write các NDJSON objects vào S3 (partition theo date/hour).
- EventBridge (cron) kích hoạt ETL Lambda (batch) theo chu kỳ cố định (ví dụ: mỗi 60 phút).
- ETL Lambda đọc một phần dữ liệu từ các partitions của S3, loại bỏ trùng lặp, chuyển đổi và upsert vào PostgreSQL trên EC2 (truy cập qua VPC).
- R Shiny Server (trên EC2) đọc các curated tables và hiển thị dashboards cho admin.

#### Data Contracts & Governance

**Event JSON (ingest)**

```yaml
{
  "event_id": "uuid-v4",
  "ts": "2025-10-18T12:34:56.789Z",
  "event_type": "view|click|search|add_to_cart|checkout|purchase",
  "session_id": "uuid-v4",
  "user_id": "cognito-sub-or-null",
  "anonymous_id": "stable-anon-id",
  "page_url": "https://site/p/123",
  "referrer": "https://google.com",
  "device": { "type": "mobile|desktop|tablet" },
  "geo": { "country": "VN", "city": null },
  "ecom":
    {
      "product_id": "sku-123",
      "category": "Shoes",
      "currency": "USD",
      "price": 79.99,
      "qty": 1,
    },
  "props": { "search_query": "running shoes" },
}
```

- **PII**: không bao giờ gửi name/email/phone; mọi optional identifier (nếu có) sẽ được hash trong Lambda.
- **Behaviour**: tạo anonymous_id một lần, duy trì session_id (tự động roll sau 30 phút không hoạt động); gửi dữ liệu bằng navigator.sendBeacon với fetch retry fallback; tùy chọn offline buffer thông qua IndexedDB.

**S3 raw layout & retention**

- **Bucket**: `s3://clickstream-raw/`
- **Object format**: NDJSON, tùy chọn GZIP.
- **Partitioning**: `year=YYYY/month=MM/day=DD/hour=HH/ → events-<uuid>.ndjson.gz`
- **Optional manifest per batch**: bao gồm processed watermark, object list, record counts, và hash.
- **Lifecycle**: raw → (30 days Standard/IA) → (365+ days Glacier/Flex).
- **Idempotency**: duy trì một compact staging table trong PostgreSQL (hoặc một small S3 key-value manifest) để theo dõi (track) last processed object/batch và ngăn chặn việc tải trùng (prevent double-load).

#### Frontend SDK (Static site on S3 + CloudFront)

**Instrumentation**

- JS snippet nhỏ được load trên toàn site (defer).
- Sinh anonymous_id một lần và lưu session_id trong localStorage; session reset sau 30 phút không hoạt động.
- Gửi events qua navigator.sendBeacon; fallback sang fetch với retry và jitter.

**Auth context**

- Nếu người dùng đăng nhập bằng Cognito, bao gồm user_id = idToken.sub để theo dõi funnel đăng nhập.

**Offline durability**

- Service Worker queue tùy chọn: khi offline, buffer events trong IndexedDB và gửi lại khi reconnect.

#### Ingestion API (API Gateway → Lambda)

**API Gateway (HTTP API)**

- Route: `POST /v1/events`.
- JWT authorizer (Cognito user pool). Đối với anonymous pre-login events, sử dụng API key với usage-plan và rate limit nghiêm ngặt.
- WAF: AWS Managed Core + Bot Control; chặn non-site origins bằng strict CORS.

**Lambda (Node.js or Python)**

- Validate theo JSON Schema (ajv/pydantic).
- Idempotency: cache `event_id` gần đây trong bộ nhớ (TTL ngắn) + dedupe ở mức batch trong ETL.
- Enrichment: xác định `date/hour`, phân tích UA, suy luận country từ `CloudFront-Viewer-Country` nếu có.
- Persist: PutObject vào đường dẫn `S3 .../year=YYYY/month=MM/day=DD/hour=HH/....`
- Failure path: publish vào SQS DLQ; cảnh báo qua SNS nếu DLQ depth > 0.

#### Batch Buffer (S3)

- Mục đích: buffer bền vững, chi phí thấp cho batch analytics.
- Write pattern: các object nhỏ mỗi request hoặc micro-batches (1–5 MB) với GZIP. Optional compactor gộp thành file ≥64MB để đọc hiệu quả hơn.
- Read pattern: ETL Lambda chỉ quét các partitions/objects mới kể từ watermark cuối.
- Schema-on-read: ETL áp dụng schema, xử lý dữ liệu đến trễ bằng cách reprocess một sliding window nhỏ (ví dụ: 2 giờ cuối) để điều chỉnh sessions.

#### EC2 “data warehouse” node

Mục đích: chạy ETL và lưu trữ analytical store được Shiny truy vấn. Hai lựa chọn:

- Postgres trên EC2 (khuyến nghị nếu nhóm quen SQL/window functions)

  - Instance: t3.small/t4g.small; gp3 50–100GB.
  - Schema: `fact_events`, `fact_sessions`, `dim_date`, `dim_product`.
  - Security: trong private subnet của VPC; truy cập qua ALB/SSM Session Manager; snapshot tự động hàng ngày lên S3.

- ETL (Lambda, batch qua EventBridge cron):
  - Trigger: rate(5 minutes) / cron(...) tùy theo cost & freshness.
  - Các bước: liệt kê S3 objects mới → đọc → validate/dedupe → transform (flatten JSON, cast types, thêm ingest_date, session_window_start/end) → upsert vào Postgres bằng COPY tới bảng tạm + merge, hoặc batched INSERT ... ON CONFLICT.
  - Networking: Lambda kết nối vào private subnets của VPC để truy cập Postgres security group trên EC2.

#### R Shiny Server on EC2 (admin analytics)

**Server**

- EC2 (t3.small/t4g.small) với: R 4.4+, Shiny Server (open-source), Nginx reverse proxy, TLS qua ACM/ALB hoặc Let’s Encrypt.
- IAM instance profile (không dùng static keys). Security group chỉ cho phép HTTPS từ office/VPN hoặc Cognito-gated admin site.

**App (packages)**

- Các package R sử dụng: `shiny`, `shinydashboard/bslib`, `plotly`, `DT`, `dplyr`, `DBI` + `RPostgres` hoặc `duckdb`, `lubridate`.
- Nếu truy vấn DynamoDB trực tiếp cho các small cards, có thể sử dụng `paws.dynamodb` (tùy chọn).

**Dashboards**

- **Traffic & Engagement**: DAU/MAU, sessions, avg pages, bounce proxy.
- **Funnels**: view → add_to_cart → checkout → purchase với tỷ lệ chuyển đổi từng stage (stage conversion) và drop-off.
- **Product Performance**: views, CTR, ATC rate, revenue theo product/category.
- **Acquisition**: referrer, campaign, device, country.
- **Reliability**: Lambda error rate, DLQ depth, ETL lag, data freshness.

**Caching**

- Kết quả truy vấn được cache trong process (reactive values) hoặc materialized bởi ETL; cache keys dựa trên date range và filters.

#### Security baseline

**IAM**

- Ingest Lambda: quyền s3:PutObject tới raw bucket (giới hạn theo prefix), s3:ListBucket trên các prefix cần thiết.
- ETL Lambda: quyền s3:GetObject/ListBucket trên raw prefixes; được phép lấy secrets từ SSM Parameter Store; không có quyền S3 rộng.
- EC2 roles: chỉ đọc/ghi vào DB/volumes của chính nó; có thể đọc từ S3 để backup.
- Shiny EC2: không ghi vào S3 raw; chỉ read-only tới Postgres khi cần.

**Network**

- Đặt EC2 trong private subnets; truy cập công khai qua ALB (HTTPS 443).
- Lambda thực hiện ETL được join vào VPC để kết nối tới Postgres; Security Group (SG) áp dụng least-privilege (chỉ mở Postgres port từ ETL SG).
- Không mở rộng 0.0.0.0/0 tới các DB ports.

**Data**

- Mã hóa: EBS bằng KMS, S3 server-side encryption, RDS/PG TLS, secrets lưu trong SSM Parameter Store.
- Dữ liệu nhạy cảm: không có PII trong events; retention: raw S3 90–365 ngày (lifecycle), curated Postgres theo chính sách kinh doanh.

#### Observability & alerting

**CloudWatch metrics/alarms**

- API Gateway 5xx/latency, Lambda (ingest) errors/throttles, S3 PutObject failures, EventBridge schedule success rate, ETL duration/lag, DLQ depth, Shiny health check.

**SNS topics**: gửi thông báo on-call qua email/SMS/Slack webhook.
**Structured logs**: JSON logs từ Lambda & ETL (bao gồm request_id, event_type, status, ms, error_code).
**Watermark tracking**: custom metric “DW Freshness (minutes since last successful upsert)”.

#### Cost Controls (stay near Free/low tier)

- HTTP API được sử dụng (chi phí thấp hơn), Lambda memory tối thiểu (256–512MB), nén requests.
- Batch thay vì realtime: dùng S3 làm buffer để loại bỏ chi phí ghi/đọc DynamoDB.
- S3 lifecycle: Standard → Standard-IA/Intelligent-Tiering → Glacier cho dữ liệu raw cũ; bật GZIP để giảm chi phí lưu trữ và truyền tải.
- Điều chỉnh cadence ETL (ví dụ: 15–60 phút) và chỉ xử lý các object mới; gộp các file nhỏ thành file lớn để giảm read I/O.
- Single small EC2 cho Shiny + DW ban đầu; sau đó scale vertically hoặc tách riêng khi cần.
- AWS Budgets với SNS alerts cho chi phí thực tế và dự báo.

#### Deliverables

- Analytics SDK (TypeScript): hỗ trợ sessionization, beacon, và optional offline queue.
- API/Lambda (ingest): xử lý validation, enrichment, idempotency hints, và DLQ.
- S3 raw bucket spec: prefixing/partitioning, compression, lifecycle, kèm optional compactor.
- ETL Lambda (batch): kết hợp với EventBridge cron, watermarking, và upsert strategy vào PostgreSQL
- PostgreSQL schema: fact_events, fact_sessions, các dims, cùng indexes và vacuum/maintenance plan.
- R Shiny dashboard app: gồm 5 modules, triển khai với Nginx/ALB TLS setup.
- Runbook: bao gồm alarms, on-call, backups, disaster recovery, freshness SLO, và cost guardrails.

### 5. Lộ trình & Mốc triển khai

### Dự án theo tiến độ

#### Tháng 1 – Học tập & Chuẩn bị

Nghiên cứu nhiều dịch vụ AWS bao gồm compute, storage, analytics và security.  
Hiểu các khái niệm chính của cloud architecture, data pipelines và serverless computing.  
Tổ chức các cuộc họp nhóm để thống nhất mục tiêu dự án và phân công trách nhiệm cho từng thành viên.

#### Tháng 2 – Thiết kế kiến trúc & Prototyping

Thiết kế kiến trúc tổng thể của dự án và xác định luồng dữ liệu giữa các thành phần.  
Thiết lập các tài nguyên AWS ban đầu như S3, Lambda, API Gateway, EventBridge và EC2.  
Thử nghiệm các công cụ mã nguồn mở cho việc visualization và reporting.  
Kiểm thử mã mẫu và xác thực quy trình data ingestion và processing pipeline.

#### Tháng 3 – Triển khai & Kiểm thử

Triển khai toàn bộ kiến trúc dựa trên bản thiết kế đã được phê duyệt.  
Tích hợp tất cả các dịch vụ AWS và đảm bảo độ tin cậy của hệ thống.  
Thực hiện kiểm thử hiệu năng và chức năng.  
Hoàn thiện tài liệu và chuẩn bị dự án cho buổi thuyết trình.

### 6. Ước tính ngân sách

Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).

### Chi phí hạ tầng

- AWS Services

  - **Amazon Cognito (User Pools)**: 0.10 USD/tháng(1 monthly active user (MAU), 1 MAU đăng nhập qua SAML hoặc OIDC federation)

  - **Amazon S3**

    - S3 Standard: 0.17 USD/tháng (6 GB, 1,000 PUT requests, 1,000 GET requests, 6 GB Data returned, 6 GB Data scanned)
    - Data Transfer: 0.00 USD/tháng (Outbound 6 TB, Inbound 6 TB)

  - **Amazon CloudFront (Asia Pacific)**: 1.08 USD/tháng(6 GB Data transfer out to internet, 6 GB Data transfer out to origin, 10,000 HTTPS requests)

  - **Amazon API Gateway (HTTP APIs)**: 0.01 USD/tháng(10,000 HTTP API requests units)

  - **Amazon Lambda (Service settings)**: 0.00 USD/tháng(1,000,000 requests, 512 MB memory)

  - **Amazon CloudWatch (APIs)**: 0.03 USD/tháng(100 metrics GetMetricData, 1,000 metrics GetMetricWidgetImage, 1,000 API requests)

  - **Amazon SNS (Service settings)**: 0.03 USD/tháng(1,000,000 requests, 100,000 HTTP/HTTPS Notifications, 1,000 EMAIL/EMAIL-JSON Notifications, 100,000,000 QS Notifications, 100,000,000 Lambda deliveries, 100,000 Kinesis Data Firehose notifications)

  - **Amazon EC2 (EC2 specifications)**: 9.42 USD/tháng(1 instance t3.small)

  - **Amazon EventBridge**: 0.00 USD/tháng(1,000,000 events (AWS management events - EventBridge Event Bus Ingestion))
  - **Amazon Amplify**: 0.20 USD/tháng(Web App Hosting)

Tổng cộng: 11.05 USD/tháng, 132.60 USD/12 tháng

### 7. Đánh giá rủi ro

| Risk                                                                                                                             | Likelihood | Impact | Mitigation Strategy                                                                                                                                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Chi phí cao vượt quá ngân sách ước tính                                                                                          | Medium     | High   | Theo dõi chặt chẽ và tính toán tất cả các chi phí tiềm năng trên AWS. Giới hạn việc sử dụng các dịch vụ AWS có chi phí cao và thay thế bằng các giải pháp đơn giản, tiết kiệm chi phí nhưng cung cấp chức năng tương tự. |
| Các vấn đề tiềm ẩn trong việc truyền dữ liệu hoặc tích hợp dịch vụ giữa các thành phần AWS                                       | Medium     | Medium | Thực hiện step-by-step validation trước khi triển khai chính thức. Thử nghiệm sớm, sử dụng các managed AWS services, và liên tục giám sát hiệu suất thông qua Amazon CloudWatch.                                         |
| Rủi ro trong thu thập hoặc xử lý dữ liệu (ví dụ: tương tác người dùng quá mức, mạng không ổn định, thiếu hoặc trùng lặp sự kiện) | High       | Medium | Áp dụng data validation, temporary buffering, và schema enforcement để đảm bảo tính nhất quán. Sử dụng structured logging và alarms để phát hiện và xử lý lỗi khi ingest dữ liệu.                                        |
| Người dùng ít hoặc không sử dụng analytics dashboard                                                                             | Low        | High   | Tổ chức các buổi internal training và tận dụng các communication channels hiện có để nâng cao nhận thức. Khuyến khích việc sử dụng bằng cách trình bày các practical benefits và actionable insights của hệ thống.       |

### 8. Kết quả kỳ vọng

#### Hiểu Hành Vi và Hành Trình Khách Hàng

Hệ thống ghi lại toàn bộ hành trình khách hàng — bao gồm các trang mà người dùng truy cập, sản phẩm mà họ xem, thời gian họ ở lại, và điểm họ rời khỏi trang web.

Bằng cách phân tích session duration, bounce rate, và navigation paths, doanh nghiệp có thể đánh giá mức độ tương tác của người dùng và trải nghiệm tổng thể.

Điều này cung cấp nền tảng dữ liệu đáng tin cậy để cải thiện giao diện website, tối ưu bố cục trang, và nâng cao overall customer satisfaction.

#### Xác Định Sản Phẩm Phổ Biến và Xu Hướng Người Tiêu Dùng

Dựa trên clickstream data được thu thập và xử lý trên AWS, hệ thống xác định các sản phẩm được xem nhiều nhất và mua nhiều nhất.

Các sản phẩm ít được chú ý cũng được theo dõi, cho phép doanh nghiệp đánh giá hiệu quả của product listings, điều chỉnh giá cả hoặc hình ảnh sản phẩm, và lập kế hoạch tồn kho hiệu quả hơn.

Hơn nữa, hệ thống hỗ trợ phát hiện shopping trends theo khoảng thời gian, khu vực, hoặc loại thiết bị — giúp đưa ra quyết định kinh doanh kịp thời và dựa trên dữ liệu.

#### Tối Ưu Chiến Lược Marketing và Bán Hàng

Dữ liệu hành vi của khách hàng được transform (chuyển đổi) thành business insights (thông tin kinh doanh chuyên sâu) và được trình bày thông qua R Shiny dashboards.

Với các kết quả phân tích này, doanh nghiệp có thể:

- Xác định chính xác target customer segments cho các nỗ lực marketing
- Tùy chỉnh các chiến dịch quảng cáo và khuyến mãi cho các nhóm sản phẩm hoặc đối tượng khách hàng cụ thể
- Đánh giá hiệu quả của các sáng kiến marketing thông qua các chỉ số tương tác và chuyển đổi có thể đo lường

Kết quả là, marketing and sales strategies trở nên dựa trên bằng chứng và chính xác hơn, hỗ trợ ra quyết định tốt hơn và cải thiện hiệu suất kinh doanh.
