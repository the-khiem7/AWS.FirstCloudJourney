---
title: "Visualizing Analytics with Shiny Dashboards"
weight: 55
chapter: false
pre: " <b> 5.5. </b> "
---

## 5.5.1 R Shiny on the Data Warehouse EC2

R Shiny is installed on `SBW_EC2_ShinyDWH` and runs alongside PostgreSQL:

- Listens on port `3838`  
- Serves the `sbw_dashboard` app  
- Connects to `clickstream_dw` via `localhost:5432`  

Typical stack:

- R + packages:
  - `shiny`, `shinydashboard`  
  - `DBI`, `RPostgres`  
  - `dplyr`, `ggplot2`, etc.  

The Shiny app is deployed under:

- `/srv/shiny-server/sbw_dashboard/`  
  - `app.R` or `ui.R` + `server.R`  

---

## 5.5.2 Dashboard Content

The dashboard can be structured into multiple tabs, for example:

1. **Overview**
   - Total events over selected time range  
   - Unique users, sessions  
   - Top event types  

2. **Product Analytics**
   - Top products by views (`event_name = 'product_view'`)  
   - Conversion to add-to-cart / checkout by product  

3. **Funnel Analysis**
   - Steps:
     - `page_view` → `product_view` → `add_to_cart` → `checkout`  
   - Drop-off rate at each step  

4. **Time-based Trends**
   - Events per hour / per day  
   - Peak user activity windows  

All of these visuals are powered by simple queries to `clickstream_dw.public.clickstream_events` and any aggregate tables you decide to build.

---

## 5.5.3 Secure Access via SSM Session Manager

Because `SBW_EC2_ShinyDWH` has no public IP and no open SSH port, access happens via **AWS Systems Manager Session Manager**.

### Steps

1. Open **Session Manager** in the AWS console.  
2. Start a session to instance `SBW_EC2_ShinyDWH`.  
3. Set up **port forwarding**:
   - `localPort = 3838`  
   - `portNumber = 3838`  

4. On your local machine, open:
   - `http://localhost:3838/sbw_dashboard`  

You can now browse the dashboard as if it were running locally, while it actually lives inside a private subnet.

---

## 5.5.4 Verifying the End-to-End Data Flow

A typical validation sequence:

1. **Generate behavior**:
   - Open the Amplify URL: `https://main.d2q6im0b1720uc.amplifyapp.com/`  
   - Navigate across product lists and product detail pages  
   - Add items to the cart, proceed to checkout  

2. **Check S3**:
   - Inspect `clickstream-s3-ingest/events/YYYY/MM/DD/`  
   - Confirm new `event-<uuid>.json` files are present  

3. **Run or wait for ETL**:
   - Trigger `SBW_Lamda_ETL` manually or wait for `SBW_ETL_HOURLY_RULE`  
   - Check CloudWatch logs to confirm rows were loaded  

4. **Query DW**:
   - From the EC2 or via a DB client:
     - `SELECT * FROM public.clickstream_events ORDER BY event_timestamp DESC LIMIT 50;`  

5. **Refresh the Shiny dashboard**:
   - Confirm charts and numbers reflect the new interactions  
