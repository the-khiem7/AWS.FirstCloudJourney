---
title: "Worklog Tuần 9"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

## Mục tiêu Tuần 9
- Hoàn thiện bước build web thương mại điện tử và bắt đầu đưa dữ liệu mẫu lên hệ thống.  
- Làm rõ vai trò của CloudFront và xác định vị trí của AWS Amplify trong kiến trúc tổng thể.  
- Thiết lập môi trường local với LocalStack để mô phỏng AWS phục vụ phát triển.  
- Thống nhất bộ thư viện Python, phiên bản và cơ chế đóng gói.  
- Tiếp tục nghiên cứu ORM và Clickstream để chuẩn bị cho phân tích dữ liệu người dùng.

---

## Các công việc cần thực hiện trong tuần

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Hoàn thiện build web. <br> - Chuẩn bị dữ liệu để đưa lên web. <br> - Tìm hiểu vai trò mở rộng của CloudFront. <br> - Làm rõ thứ tự CloudFront – Amplify trong kiến trúc. <br> - Thiết lập LocalStack để test AWS offline. <br> - Thống nhất thư viện Python và phiên bản. <br> - Xây dựng ORM và tiếp tục nghiên cứu Clickstream đến ngày 07/11. | 05/11/2025 14:15 | 05/11/2025 15:40 | Internal Notes |
| 2 | - Xem lại CloudFront và tối ưu. <br> - Chọn LocalStack bản BASE để hỗ trợ Cognito. <br> - Sửa website, chuyển database sang Supabase. <br> - Điều chỉnh lại luồng kiến trúc CloudFront + Amplify. <br> - Cập nhật kiến trúc: thêm Amplify và PostgreSQL. <br> - Hoàn thiện danh sách thư viện Python. <br> - Cập nhật proposal theo mục tiêu dự án. | 09/11/2025 20:30 | 09/11/2025 00:00 | Internal Notes |

---

## Thành tựu Tuần 9

### 1. Hoàn thiện build web và bắt đầu xử lý dữ liệu
- Web thương mại điện tử đã được build thành công.  
- Bắt đầu đưa dữ liệu mẫu vào hệ thống để kiểm thử luồng sản phẩm – giỏ hàng – người dùng.  
- Chuyển từ giai đoạn khởi tạo sang chạy với dữ liệu thực tế.

### 2. Hiểu rõ vai trò của CloudFront trong kiến trúc
- CloudFront không chỉ tăng tốc mà còn tăng tính bảo mật, che hạ tầng và tối ưu phân phối.  
- Team xác định rõ Amplify đứng sau CloudFront trong kiến trúc triển khai.  
- Tạo nền tảng đúng chuẩn cho pipeline deploy về sau.

### 3. Thiết lập LocalStack làm môi trường mô phỏng AWS
- Quyết định sử dụng LocalStack BASE để hỗ trợ Cognito đầy đủ.  
- Môi trường local giúp test AWS mà không tốn chi phí và giảm rủi ro khi deploy thật.  
- Hỗ trợ phát triển nhanh, thử nghiệm CloudFront–S3–Cognito offline.

### 4. Chuẩn hóa môi trường Python
- Thống nhất bộ thư viện Python và phiên bản sử dụng chung.  
- Tránh lỗi không tương thích giữa các thành viên khi chạy script hoặc xử lý Clickstream.  
- Tạo nền cho việc đóng gói, deploy automation sau này.

### 5. Điều chỉnh kiến trúc hệ thống
- Kiến trúc được cập nhật rõ ràng hơn:  
  - Amplify phía sau CloudFront  
  - PostgreSQL/Supabase làm database  
- Giúp hệ thống có hướng đi rõ ràng, phù hợp mô hình serverless.

### 6. Sửa website và tích hợp Supabase
- Database chuyển sang Supabase, dễ kết nối và phát triển nhanh.  
- Frontend đã kết nối ổn định với database mới.  
- Giảm thời gian setup so với tự dựng DB.

### 7. Nghiên cứu Clickstream và ORM
- Tìm hiểu cơ chế tracking hành vi người dùng, pipeline sự kiện.  
- ORM được đọc và hiểu để chuẩn bị cho database thật.  
- Là nền tảng cho các tuần tập trung vào phân tích dữ liệu và thống kê.

## Tổng kết Tuần 9
Tuần 9 củng cố kiến trúc, hoàn thiện môi trường phát triển, build web thành công và bắt đầu tích hợp dữ liệu.  
Đây là bước quan trọng để bước vào giai đoạn tăng tốc triển khai trên AWS thực tế trong các tuần tiếp theo.
