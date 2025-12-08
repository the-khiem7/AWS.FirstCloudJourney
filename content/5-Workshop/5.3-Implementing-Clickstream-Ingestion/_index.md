---
title: "Implementing Clickstream Ingestion"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

This section describes in detail how clickstream events are generated in the browser, sent to **Amazon API Gateway**, processed by the **Lambda Ingest** function, and finally stored as raw JSON files in the **S3 Raw Clickstream bucket**.

The ingestion path is the **entry point of the entire analytics platform**: if events are not captured or stored correctly at this stage, the downstream ETL and dashboards will not be reliable.

---

### Frontend event generation

The e-commerce frontend is implemented using **Next.js** and hosted on **AWS Amplify**. A client-side tracking module is responsible for collecting user actions and sending them as clickstream events.

Typical actions that should be captured include:

- **Page views**: landing page, category pages, search results, product detail pages.  
- **Product interactions**: clicking on a product card, viewing product details, adding or removing items from the cart.  
- **Checkout steps**: starting the checkout, filling in shipping information, placing an order.  
- **Session and identity information**: user ID (if logged in), session ID, login state, client ID.

The frontend tracking code usually performs the following steps:

1. Listens for events in the UI (e.g., page load, button click).  
2. Builds a JSON payload with at least:

   - `event_id` – unique identifier for the event.  
   - `event_name` – e.g., `"page_view"`, `"product_view"`, `"add_to_cart"`.  
   - `event_timestamp` – client-side timestamp (ISO 8601 or epoch).  
   - `user_id` / `identity_source` – if the user is logged in (e.g., Cognito user sub).  
   - `session_id` / `client_id` – session or browser identifier.  
   - `product_id`, `product_name`, `product_category`, etc., for product-related events.  
   - `page_url`, `referrer`, `utm_*` fields for traffic attribution (optional).

3. Sends the JSON payload via HTTPS to the API Gateway endpoint (see Section 5.3.2).

Example of a minimal JSON payload sent from the browser:

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

The payload can be extended later as the analytics requirements evolve.

---

### API Gateway and Lambda Ingest flow

#### API Gateway HTTP API

The ingestion endpoint is exposed using **Amazon API Gateway (HTTP API)** with a route such as:

- Method: `POST`  
- Path: `/clickstream`  

Key characteristics:

- The endpoint accepts JSON payloads from the browser.  
- CORS is configured to allow the frontend domain (CloudFront / Amplify) to call it.  
- The route is integrated with a **Lambda function** (Lambda proxy integration).
  
**Figure 5-5: API Gateway route for POST /clickstream**

The screenshot shows the HTTP API (`clickstream-http-api`) configuration for the clickstream ingestion endpoint.  
The `/clickstream` resource exposes a single `POST` route, which is integrated with the `clickstream-lambda-ingest` Lambda function.  
No authorizer is attached in this lab to keep the workshop focused on data ingestion rather than authentication.

![Figure 5-5: API Gateway route for POST /clickstream](/images/5-3-apigw-clickstream-route.png)


#### Lambda Ingest function

The **Lambda Ingest** function is responsible for the following:

1. **Parsing the request**  
   - Reads the request body from the API Gateway event.  
   - Validates that the body contains a valid JSON object or array of objects.

2. **Basic validation & enrichment**  
   - Ensures mandatory fields (e.g., `event_name`, `event_timestamp`) are present.  
   - If the client timestamp is missing, attaches a server-side timestamp.  
   - Adds metadata such as:
     - The API Gateway request ID.  
     - The source IP or user agent (if needed).  

3. **Batching and writing to S3**  
   - Constructs a key in the Raw Clickstream bucket (`clickstream-s3-ingest`) using a time-based partition pattern, for example:

     ```text
     s3://clickstream-s3-ingest/events/YYYY/MM/DD/event-<uuid>.json
     ```

   - Writes the incoming events as a JSON array or NDJSON (newline-delimited JSON), depending on the chosen format.

4. **Logging and error handling**  
   - Logs validation errors or malformed events to **CloudWatch Logs**.  
   - Returns an appropriate HTTP status code back to API Gateway (e.g., `200` on success, `400` for invalid payload).

Example S3 object key for an event batch captured on 4 December 2025:

```text
events/2025/12/04/event-a1b2c3d4.json
```

**Figure 5-6: Lambda Ingest function overview**

The screenshot shows the `clickstream-lambda-ingest` function in the AWS Lambda console.  
The diagram highlights that the function is triggered by the HTTP API Gateway and uses a lightweight configuration (128 MB memory, short timeout) that is sufficient for validating JSON payloads, enriching metadata, and writing batched events to the S3 Raw Clickstream bucket.

![Figure 5-6: Lambda Ingest function overview](/images/5-3-lambda-ingest-config.png)

---

### Manual testing from the frontend and Postman

To confirm that ingestion is working as expected, two complementary tests are performed.

#### End-to-end test from the real frontend

1. Open the Amplify app domain of the e-commerce application:

   ```text
   https://main.d2q6im0b1720uc.amplifyapp.com/
   ```

2. Sign in using **Amazon Cognito** with a test user account.  
3. Perform a sequence of actions, such as:

   - Open the home page and a category page.  
   - View the details of two or three products.  
   - Add at least one product to the cart and then remove it.  
   - Start the checkout flow and proceed to the final confirmation step (placing a real order is optional).

4. Using the browser’s developer tools (Network tab), verify that:

   - Requests are being sent to `POST /clickstream`.  
   - The request body contains the expected JSON fields such as `event_name`, `product_id`, `session_id`, etc.  
   - The response status code from API Gateway is `200`.

#### Direct test using Postman or Thunder Client

For controlled testing, you can also send synthetic events using an API client:

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

This approach is helpful to test error handling, validation rules, or edge cases without depending on the frontend code.

---

### Inspecting data in the S3 Raw Clickstream bucket

Once events have been sent, the next step is to verify that they were successfully stored in S3.

1. Open the **Amazon S3 Console** and select the Raw Clickstream bucket, for example:

   ```text
   clickstream-raw-<account>-<region>
   ```

2. Navigate through the prefix structure:

   ```text
   events/YYYY/MM/DD/HH/
   ```

   For example:

   ```text
   events/2025/12/04/15/
   ```

3. Confirm that one or more objects with names similar to:

   ```text
   events-a1b2c3d4.json
   events-f9e8d7c6.json
   ```

   have been created.

4. Download one of these files and open it with a text editor (VS Code, Notepad++, etc.):

   - Verify that the JSON structure is valid.  
   - Check that fields such as `event_name`, `event_timestamp`, `user_id`, `session_id`, `product_id`, etc., match the actions you performed on the website.  
   - Ensure that multiple events are grouped logically (for example, as an array of JSON objects).

5. Take note of:

   - The approximate file size (KB/MB).  
   - The number of events per file.  
   - How frequently new files appear (depending on how the Ingest Lambda batches writes).

These observations will help when tuning ETL batch sizes and scheduling in later sections.

**Figure 5-7: Raw clickstream JSON objects in the S3 bucket**

The screenshot shows the S3 Raw Clickstream bucket at the deepest hour-level prefix.  
Multiple JSON objects have been created by the Lambda Ingest function, each representing a batch of clickstream events captured during that hour.  
The file names follow a UUID-based pattern (for example, `event-a8bdf0c0-...json`), and the `Size` column gives a quick indication of how many events are grouped in each file.

![Figure 5-7: Raw clickstream JSON objects in the S3 bucket](/images/5-3-s3-raw-objects.png)

---

### Summary and relation to downstream components

At the end of this section, you should have:

- A working ingestion path from **browser → API Gateway → Lambda Ingest → S3 Raw bucket**.  
- Verified that events are correctly structured and contain the necessary metadata.  
- Confirmed that the Raw Clickstream bucket uses a **time-partitioned layout** that is suitable for batch processing.

In the next section (5.4), the focus will shift to the **private analytics layer**, where a VPC-enabled **ETL Lambda** reads these raw JSON files via an **S3 Gateway VPC Endpoint**, transforms them, and loads them into the **PostgreSQL Data Warehouse**.

---

**Figure 5-8: Ingestion flow from Browser to S3 Raw bucket**

The diagram illustrates the sequence:

- Browser (user actions) → **CloudFront** → **Amplify** (Next.js app).  
- Frontend tracking sends HTTP requests to **Amazon API Gateway (HTTP API)** on the `POST /clickstream` route.  
- API Gateway invokes **Lambda Ingest**, which validates and enriches events.  
- Lambda Ingest writes **batched JSON objects** into the **S3 Raw Clickstream bucket** under a time-partitioned key such as `events/YYYY/MM/DD/HH/events-<uuid>.json`.

![Figure 5-8: Ingestion flow from frontend to S3 Raw bucket](/images/5-3-ingestion-flow.png)
