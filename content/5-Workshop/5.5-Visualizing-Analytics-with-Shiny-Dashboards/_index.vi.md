---
title: "Visualizing Analytics with Shiny Dashboards"
weight: 55
chapter: false
pre: " <b> 5.5. </b> "
---

## 5.5.1 Thông tin môi trường

- OS: **Ubuntu 22.04 (Jammy)** – EC2 trong private subnet  
- PostgreSQL: **v18** (cài từ repo `apt.postgresql.org`)  
- Shiny Server: bản binary `.deb` từ RStudio (Posit)  
- User chạy Shiny: `shiny`  
- Đường dẫn app: `/srv/shiny-server/sbw_dashboard/app.R`

---

## 5.5.2 Cài các package hệ thống (system libs)

Đăng nhập EC2 bằng **SSM Session Manager** hoặc SSH (tạm thời, nếu có), sau đó chạy:

```bash
# 1) Update danh sách package
sudo apt-get update

# 2) Cài R (nếu chưa cài)
sudo apt-get install -y r-base

# 3) Cài Postgres client & dev headers (cho RPostgres)
#    Nếu DB của bạn là PG 18 thì dùng postgresql-server-dev-18
#    (nếu version khác thì đổi số 18 -> 14, 15, ...)
sudo apt-get install -y postgresql-client-18 postgresql-server-dev-18

# 4) Cài libpq + libssl (bắt buộc để build RPostgres)
sudo apt-get install -y libpq-dev libssl-dev

# 5) (Nếu chưa cài Shiny Server)
#    Tùy theo cách bạn đã cài, ở đây chỉ ghi nhớ:
#    - shiny-server service: /etc/systemd/system/shiny-server.service
#    - thư mục app: /srv/shiny-server/
#    - user chạy: shiny
```

Kiểm tra lại `libpq` và dev headers đã có:

```bash
dpkg -l | grep -E 'libpq-dev|postgresql-server-dev' || echo "MISSING_LIBS"
ls -l /usr/include/postgresql/libpq-fe.h || echo "NO_LIBPQ_HEADER"
```

Nếu **không thấy lỗi** → OK.

---

## 5.5.3 Cấu hình thư mục R libraries cho user `shiny`

Để Shiny Server load được các package R, ta cài package dưới user `shiny` và dùng thư mục:

- `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`

Chạy:

```bash
sudo -u shiny R --vanilla <<'EOF'
# Tạo thư mục library cho user shiny nếu chưa có
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)

# Đưa R_LIBS_USER lên đầu .libPaths()
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

q("no")
EOF
```

Bạn sẽ thấy `LIBPATHS` có dòng 1 là `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`.

---

## 5.5.4 Cài các R package cần thiết

Các package cần cho dashboard:

- `shiny`
- `DBI`
- `RPostgres`
- `dplyr`
- `ggplot2`
- `lubridate`
- `pool`

Cài tất cả dưới user `shiny`:

```bash
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)

q("no")
EOF
```

💡 **Nếu gặp lỗi liên quan tới `libpq-fe.h` hoặc `libpq`:**

1. Kiểm tra lại đã cài `libpq-dev`, `postgresql-server-dev-XX`, `libssl-dev` chưa.  
2. Chạy lại `install.packages("RPostgres", ...)` sau khi cài đủ libs.  

Kiểm tra lại việc load package:

```bash
sudo -u shiny R --vanilla <<'EOF'
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

library(shiny)
library(DBI)
library(RPostgres)
library(dplyr)
library(ggplot2)
library(lubridate)
library(pool)

cat("All packages loaded OK
")
q("no")
EOF
```

Nếu **không có error** → môi trường R đã OK.

---

## 5.5.5 Triển khai Shiny app

### 5.5.5.1 Tạo thư mục app và copy code

```bash
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
```

Tạo (hoặc thay) file app:

```bash
sudo nano /srv/shiny-server/sbw_dashboard/app.R
# DÁN TOÀN BỘ CODE app.R (bản full mà bạn đang dùng)
# Ctrl+O, Enter, Ctrl+X để lưu
```

Đảm bảo quyền:

```bash
sudo chown shiny:shiny /srv/shiny-server/sbw_dashboard/app.R
sudo chmod 644 /srv/shiny-server/sbw_dashboard/app.R
```

### 5.5.5.2 Restart Shiny Server

```bash
sudo systemctl restart shiny-server
sudo systemctl status shiny-server
```

---

## 5.5.6 Kiểm tra app từ EC2 (local)

Từ session SSM trên EC2 (terminal):

```bash
# Check trang welcome Shiny
curl -m 5  -sS -o /dev/null -w "WELCOME HTTP %{http_code}
"   http://127.0.0.1:3838/

# Check app SBW dashboard
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```

Nếu trả về `DASHBOARD HTTP 200` → app chạy OK.

Nếu trả về `500`:

```bash
LATEST=$(ls -1t /var/log/shiny-server/sbw_dashboard-shiny-*.log | head -n 1)
echo "LATEST=$LATEST"
sudo tail -n 100 "$LATEST"
```

Xem error log để debug.

---

## 5.5.7 Truy cập dashboard từ máy local

Vì EC2 ở **private subnet**, bạn dùng **SSM port forwarding**:

```bash
# Ví dụ dùng AWS CLI v2 trên máy local:
aws ssm start-session   --target <INSTANCE_ID_PRIVATE>   --document-name AWS-StartPortForwardingSessionToRemoteHost   --parameters '{"host":["127.0.0.1"],"portNumber":["3838"],"localPortNumber":["3838"]}'
```

Sau đó, trên máy local mở trình duyệt tới:

```text
http://127.0.0.1:3838/sbw_dashboard/
```

Dashboard sẽ hiển thị với, ví dụ:

- Các **KPI cards** (tổng số events, users, sessions…)  
- Biểu đồ **events over time**, **event mix**, **events by login state**  
- Tab **Products & Raw sample** (phân trang, newest trước, auto refresh mỗi 10s – tuỳ code app của bạn)

---

## 5.5.8 Tóm tắt nhanh các lệnh quan trọng

```bash
# Cài system libs
sudo apt-get update
sudo apt-get install -y r-base postgresql-client-18 postgresql-server-dev-18 libpq-dev libssl-dev

# Cài R packages cho user shiny
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)
q("no")
EOF

# Deploy app
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo nano /srv/shiny-server/sbw_dashboard/app.R   # dán code
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
sudo systemctl restart shiny-server

# Kiểm tra dashboard
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/


