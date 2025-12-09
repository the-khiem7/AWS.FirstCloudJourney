---
title: "Worklog Tuần 12"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Mục tiêu Tuần 12
- Hoàn thiện môi trường mô phỏng AWS bằng LocalStack ở mức chuyên sâu.  
- Thực hiện triển khai hạ tầng AWS bằng Terraform trên cả môi trường LocalStack và AWS thật.  
- Tích hợp NextJS vào Amplify trong cả local và cloud.  
- Hoàn thiện mô hình dữ liệu (dataframe) và chuẩn hóa cấu trúc database.  
- Đẩy mạnh việc deploy web lên Amplify, CloudFront và kiểm tra hoạt động toàn hệ thống.  
- Chuẩn bị cho giai đoạn vận hành, logging và monitor (CloudWatch, SNS).  

---

## Các công việc cần thực hiện trong tuần

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Học LocalStack nâng cao (Duy Khiêm chủ đạo). <br> - Hoàn tất LocalStack Pro: config API Key, chạy docker image, apply Terraform vào LocalStack. <br> - Terraform: sửa TF theo diagram, kích hoạt CloudFront trong Amplify. <br> - Amplify: nghiên cứu cách đưa NextJS vào Amplify local. <br> - Cognito: implement vào môi trường Amplify local. <br> - Web data: deploy PostgreSQL lên EC2, đẩy ảnh vào S3. | 24/11/2025 20:00 | 24/11/2025 22:00 | Internal Notes |
| 2 | - Chạy Terraform. <br> - Nghiên cứu khả năng LocalStack có thể host UI và tác vụ gì được đưa lên UI. <br> - Tạo project mới chạy Amplify trên LocalStack để test. <br> - LocalStack: tạo Terraform riêng cho Amplify, truyền NextJS vào Amplify, kích hoạt CloudFront & S3 integration. <br> - AWS thật: tạo instance Amplify, push project, bật CloudFront, bật S3 integration. <br> - Tìm hiểu cách deploy Terraform lên AWS thật và quy trình xử lý lỗi. <br> - Học cách các Solutions Architect triển khai cloud hiệu quả. | 30/11/2025 15:00 | 01/12/2025 01:30 | Internal Notes |
| 3 | - Triển khai web lên Amplify (AWS thật). <br> - Kích hoạt CloudFront từ Amplify. <br> - Kiểm tra khả năng tạo S3 chuyên biệt cho Amplify (nếu có thể, tiến hành). <br> - Tích hợp Cognito vào web. <br> - Thiết kế dataframe lớn và tách thành 5 bảng. <br> - Terraform deploy các dịch vụ lên AWS cloud. <br> - Tạo các dịch vụ AWS cloud trên Amazon console để đối chiếu Terraform. | 01/12/2025 22:30 | 02/12/2025 03:00 | Internal Notes |
| 4 | - Phân công nhiệm vụ: tìm hiểu CloudWatch, SNS, tạo S3 lưu ảnh, làm SDK bắn event, tạo VPC–Subnet–IGW–EC2–Endpoint. | 02/12/2025 | 02/12/2025 | Internal Notes |
| 5 | - Hoàn thiện phần còn lại của dự án. <br> - Fix lỗi NAN trong bảng database. <br> - Vẽ biểu đồ phân tích. <br> - Khiêm làm workshop bản tiếng Anh. <br> - Thống làm workshop bản tiếng Việt. <br> - Hào upload code Shiny lên Git. <br> - Workshop phải mô tả khó khăn + giải pháp. | 07/12/2025 00:00 | 08/12/2025 00:00 | Internal Notes |

---

## Thành tựu Tuần 12

### 1. Làm chủ LocalStack Pro và mô phỏng AWS ở mức cao
- Team đã hoàn thiện LocalStack Pro: chạy bằng Docker, cấu hình API Key, triển khai Terraform trực tiếp lên LocalStack.  
- Giúp mô phỏng hầu hết dịch vụ AWS: Amplify, CloudFront, S3, Cognito, EC2, VPC… ngay trên local.  
- Đây là bước đột phá, giảm chi phí AWS thật trong quá trình thử nghiệm.

### 2. Terraform triển khai thành công ở cả local và cloud
- Terraform được điều chỉnh khớp với kiến trúc và đã chạy được trên LocalStack.  
- Sau đó team triển khai Terraform lên AWS thật, học cách xử lý lỗi, rollback và tối ưu.  
- Đây là nền tảng quan trọng cho CI/CD hạ tầng và kỹ năng DevOps thực chiến.

### 3. Tích hợp NextJS vào Amplify (local + AWS thật)
- Thành công đưa NextJS vào Amplify trên LocalStack.  
- Tiếp tục triển khai trên AWS thật, kích hoạt CloudFront và S3 integration.  
- Kết quả: web chạy được trên Amplify với pipeline hoàn chỉnh.

### 4. Hoàn thiện mô hình dữ liệu và database
- Thiết kế dataframe lớn và tách thành 5 bảng tối ưu.  
- PostgreSQL được deploy lên EC2 cho môi trường cloud.  
- S3 được dùng để lưu ảnh sản phẩm, chuẩn hóa quy trình upload dữ liệu.

### 5. Triển khai web e-commerce lên AWS Amplify
- Website được deploy lên Amplify trên AWS thật, CloudFront được bật để phân phối.  
- Kiểm tra khả năng tạo S3 chuyên biệt cho Amplify → thực hiện nếu hệ thống hỗ trợ.  
- Cognito được gắn hoàn chỉnh vào web, hoạt động ổn định.

### 6. Xây dựng kỹ năng thực tế về vận hành và triển khai
- Team học cách deploy Terraform lên AWS, xử lý lỗi, tối ưu pipeline.  
- Tìm hiểu cách Solutions Architect tư duy, lập kế hoạch triển khai và kiểm soát chi phí.  
- Đây là kỹ năng thực tế giá trị cao, giúp team sẵn sàng bước qua giai đoạn vận hành.

### 7. Hoàn thiện workshop và nội dung báo cáo
- Khiêm hoàn thành workshop tiếng Anh, Thống hoàn thành bản tiếng Việt.  
- Nội dung workshop mô tả đầy đủ khó khăn, bài học và hướng cải thiện.  
- Code Shiny được Hào upload lên Git.

### 8. Sửa lỗi hệ thống và hoàn thiện dashboard
- Fix lỗi NAN trong database.  
- Vẽ các biểu đồ phục vụ phân tích hành vi và tổng hợp dữ liệu dự án.  
- Hoàn tất để bàn giao phần phân tích dữ liệu.

## Tổng kết Tuần 12
Tuần 12 là tuần quan trọng nhất, đưa toàn bộ hệ thống từ môi trường mô phỏng sang AWS thật.  
Team đã triển khai được Amplify – CloudFront – S3 – Cognito bằng Terraform, hoàn thiện database, mô phỏng AWS bằng LocalStack Pro và hoàn tất workshop.

Đây là tuần đánh dấu dự án bước sang giai đoạn “production-ready”.