---
title: "Worklog Tuần 11"
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

## Mục tiêu Tuần 11
- Tiếp tục củng cố kiến trúc hệ thống và quy trình triển khai.  
- Hoàn thiện template tài liệu và kiểm tra chất lượng nội dung.  
- Nghiên cứu và kiểm thử website thương mại điện tử trên môi trường thực tế.  
- Khởi động học Docker và LocalStack chuyên sâu hơn nhằm chuẩn bị cho triển khai AWS thật.  
- Kích hoạt các công cụ hỗ trợ phát triển: GitHub Student, Docker, LocalStack.  

---

## Các công việc cần thực hiện trong tuần

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Kiểm tra template docx. <br> - Chạy và nghiên cứu dự án. <br> - Kiểm tra ổn định của website bán hàng. <br> - Sử dụng LocalStack để test dịch vụ AWS. <br> - Kích hoạt GitHub Student. <br> - Học cách sử dụng Docker và LocalStack. | 21/11/2025 12:00 | 21/11/2025 14:00 | Internal Notes |
| 2 | - Test website thương mại điện tử. <br> - Kiểm tra luồng xem sản phẩm, thêm giỏ hàng, mua hàng. <br> - Thử nghiệm đăng nhập bằng Clerk → thất bại → chuyển sang Cognito. <br> - Deploy web lên Vercel (4 lần thất bại, lần 5 thành công). <br> - Sửa next.config.ts, package.json, remove duplicate configs, fix eslint. <br> - Thêm file .env. <br> - Lưu được userId vào database bằng userpool. | 23/11/2025 23:00 | 24/11/2025 01:00 | Internal Notes |

---

## Thành tựu Tuần 11

### 1. Cải thiện chất lượng template tài liệu
- Template docx đã được kiểm tra và tinh chỉnh lại để phù hợp với format báo cáo toàn dự án.  
- Những lỗi bố cục, căn chỉnh và trình bày đã được sửa lại để dễ đọc, dễ theo dõi hơn.  
- Đây là nền tảng quan trọng cho các tài liệu chính thức trong tuần 12 và khi báo cáo workshop.

### 2. Kiểm thử và đánh giá website thương mại điện tử
- Website được đưa vào test chức năng thực tế: xem sản phẩm, thêm giỏ hàng, luồng mua hàng…  
- Giúp đánh giá tính ổn định của toàn hệ thống và xác định các lỗi UI/UX còn tồn tại.  
- Là bước chuyển quan trọng từ demo sang sản phẩm thực tế.

### 3. Chuyển đổi từ Clerk sang Cognito cho tính năng đăng nhập
- Đăng nhập bằng Clerk không thể lấy userId về database → giải pháp thất bại.  
- Team quyết định chuyển toàn bộ sang **AWS Cognito**, phù hợp với kiến trúc AWS.  
- Quyết định này giúp đồng bộ hóa hệ thống auth, dễ quản lý và phù hợp với serverless.

### 4. Deploy web thành công lên Vercel
- Sau 4 lần thất bại liên quan đến config, lần thứ 5 deploy thành công.  
- Các lỗi đã được xử lý: next.config trùng lặp, lỗi eslint, cấu hình package sai.  
- Đây là dấu mốc quan trọng, giúp website có thể public và hoạt động giống sản phẩm thật.

### 5. Lưu userId vào database thành công
- Website đã lấy được userId từ Cognito và lưu vào Supabase/PostgreSQL.  
- Điều này giúp chuẩn bị cho các chức năng nâng cao: quản lý đơn hàng, lịch sử giao dịch, phân tích hành vi.

### 6. Làm chủ Docker và LocalStack
- Team đã học cách hoạt động của Docker để đóng gói môi trường.  
- LocalStack được chạy ổn định giúp mô phỏng AWS mà không tốn chi phí.  
- Kiến thức này cực kỳ quan trọng cho tuần 12, khi team bắt đầu triển khai AWS thật.

### 7. Kích hoạt GitHub Student
- Hỗ trợ nhiều dịch vụ miễn phí quan trọng cho phát triển dự án (credits, công cụ, cloud…).  
- Tăng khả năng thử nghiệm và mở rộng mà không bị giới hạn tài chính.

## Tổng kết Tuần 11
Tuần 11 tập trung mạnh vào kiểm thử hệ thống, ổn định tài liệu và chuẩn hóa các công cụ phát triển.  
Website đã chạy được trên Vercel, auth hoạt động ổn định với Cognito và môi trường phát triển đã được chuẩn bị đầy đủ để bước sang triển khai cloud trong Tuần 12.
