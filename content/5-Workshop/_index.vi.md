---
title: "Workshop"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Đảm bảo truy cập Hybrid an toàn đến S3 bằng cách sử dụng VPC endpoint

#### Tổng quan

**AWS PrivateLink** cung cấp kết nối riêng tư, khả năng mở rộng cao và bảo mật giữa VPC của bạn với các dịch vụ AWS được hỗ trợ, cũng như với các dịch vụ do bên thứ ba hoặc chính bạn cung cấp. Lưu lượng luôn đi trên hạ tầng mạng nội bộ của AWS thay vì Internet công cộng, giúp giảm bề mặt tấn công và đơn giản hóa bài toán bảo mật.

Trong workshop này, bạn sẽ học cách thiết kế và triển khai một kiến trúc **hybrid** cho phép truy cập an toàn đến **Amazon S3** từ:

- Các workload chạy trong VPC (ứng dụng, batch job, analytic job,…).  
- Hạ tầng tại chỗ (on‑premises) kết nối lên AWS thông qua **AWS Direct Connect** hoặc **VPN**.

Bạn sẽ thực hành tạo, cấu hình và kiểm thử các loại **VPC endpoint** khác nhau để truy cập S3 mà **không cần dùng Internet Gateway hoặc NAT Gateway**.

Cụ thể, chúng ta sẽ làm việc với **hai loại endpoint** để truy cập Amazon S3:

- **Gateway endpoint** – Cho phép VPC gửi lưu lượng đến Amazon S3 bằng cách thêm thành phần “điểm cuối” vào **route table** của các subnet. Lưu lượng từ subnet tới S3 sẽ được định tuyến nội bộ thông qua endpoint này mà không đi qua Internet.  
- **Interface endpoint** – Tạo một hoặc nhiều **Elastic Network Interface (ENI)** trong VPC, mỗi ENI có private IP và được liên kết với một dịch vụ được hỗ trợ thông qua **AWS PrivateLink**. Lưu lượng tới S3 sử dụng endpoint này sẽ được resolve qua DNS và đi hoàn toàn trên mạng riêng của AWS, kể cả khi được truy cập từ môi trường on‑premises thông qua Direct Connect hoặc VPN.

Sau khi hoàn thành workshop, bạn sẽ hiểu rõ sự khác nhau giữa hai loại endpoint, thời điểm nên sử dụng từng loại, cũng như các tác động về bảo mật và chi phí khi thiết kế đường truy cập hybrid đến S3.

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Truy cập đến S3 từ VPC](5.3-S3-vpc/)
4. [Truy cập đến S3 từ TTDL On-premises](5.4-S3-onprem/)
5. [VPC Endpoint Policies (làm thêm)](5.5-Policy/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)
