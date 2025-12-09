---
title: "Worklog Tuần 10"
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

## Mục tiêu Tuần 10
- Hoàn thiện và tinh chỉnh lại kiến trúc hệ thống dựa trên CloudFront, Amplify và Supabase.  
- Thử nghiệm khả năng deploy local bằng LocalStack và chuẩn bị cho deploy thực tế lên AWS.  
- Điều chỉnh website và nâng cấp database theo hướng ổn định hơn.  
- Phát triển tài liệu dự án và chuẩn hóa nội dung tiếng Việt – tiếng Anh.  
- Hoàn thiện template tài liệu (docx) để đưa lên Hugo phục vụ trình bày.  

---

## Các công việc cần thực hiện trong tuần

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Rà soát lại CloudFront. <br> - Xác nhận dùng LocalStack bản BASE để hỗ trợ Cognito. <br> - Sửa website, chuyển database sang Supabase. <br> - Điều chỉnh lại flow kiến trúc CloudFront – Amplify. <br> - Cập nhật kiến trúc, thêm Amplify và PostgreSQL. <br> - Hoàn thiện danh sách thư viện và phiên bản Python. <br> - Chỉnh sửa mục tiêu trong proposal. | 09/11/2025 20:30 | 09/11/2025 00:00 | Internal Notes |
| 2 | - Sử dụng Docker để tìm hiểu môi trường cài đặt thư viện tương thích LocalStack. <br> - Chỉnh sửa lại kiến trúc hệ thống. <br> - Chỉnh sửa bản tài liệu tiếng Việt. <br> - Làm file template docx để đưa lên Hugo (chưa hoàn thành). | 11/11/2025 20:00 | 11/11/2025 23:30 | Internal Notes |
| 3 | - Chỉnh lại kiến trúc (v9). | 12/11/2025 20:00 | 12/11/2025 23:00 | Internal Notes |
| 4 | - Hoàn thiện template. <br> - Chia nội dung cho từng thành viên: Hào (2), Tín (3), Anh (4), Khiêm (5), Thống (6), AI (7). <br> - Deadline: 14h ngày 16/11/2025. | 13/11/2025 20:00 | 13/11/2025 21:30 | Internal Notes |
| 5 | - Kiểm tra nội dung template và duyệt hoàn chỉnh. | 16/11/2025 13:00 | 16/11/2025 14:00 | Internal Notes |

---

## Thành tựu Tuần 10

### 1. Củng cố và hoàn thiện kiến trúc hệ thống
- Kiến trúc hệ thống được cập nhật qua nhiều phiên bản, đến tuần này đã hoàn thiện phiên bản v9.  
- Các thành phần CloudFront, Amplify, Supabase/PostgreSQL đã được sắp xếp lại hợp lý, phản ánh đúng luồng vận hành của hệ thống.  
- Định hình rõ mô hình serverless và cách tích hợp auth, storage, CDN, database.

### 2. Hoàn thiện quy trình phát triển với LocalStack và Docker
- LocalStack bản BASE được lựa chọn để đảm bảo chạy Cognito – dịch vụ thường không hỗ trợ ở bản thường.  
- Docker được nghiên cứu và áp dụng để đóng gói môi trường, đảm bảo việc chạy LocalStack đồng nhất giữa các thành viên.  
- Đây là bước tiến quan trọng giúp mô phỏng AWS đầy đủ, giảm sai số khi deploy lên AWS thật.

### 3. Điều chỉnh website và chuyển database sang Supabase
- Database được chuyển sang Supabase giúp tốc độ phát triển nhanh hơn, dễ quản lý và đơn giản hóa việc tích hợp.  
- Website được điều chỉnh lại logic để phù hợp với cơ sở dữ liệu mới.  
- Kiến trúc dữ liệu ổn định hơn và chuẩn bị tốt cho phân tích người dùng.

### 4. Chuẩn hóa tài liệu – tiếng Việt và tiếng Anh
- Nội dung tài liệu tiếng Việt được chỉnh sửa để nhất quán, dễ đọc hơn.  
- Đây là bước quan trọng để chuyển đổi sang bản tiếng Anh sau này, phục vụ workshop hoặc báo cáo chính thức.

### 5. Xây dựng file template docx và xuất bản lên Hugo
- File template đã được tạo để đưa lên Hugo làm bộ khung báo cáo.  
- Phần này dù chưa hoàn thiện hoàn toàn nhưng đã có cấu trúc đầy đủ để các tuần tiếp theo bổ sung.

### 6. Phân công nội dung chi tiết cho từng thành viên
- Team được chia công việc rõ ràng theo từng phần tài liệu.  
- Việc phân trách nhiệm theo mục nội dung giúp đẩy nhanh tiến độ và giảm trùng lặp.

## Tổng kết Tuần 10
Tuần 10 tập trung vào việc củng cố kiến trúc, chuẩn hóa tài liệu và hoàn thiện môi trường phát triển.  
Kiến trúc hệ thống đã rõ ràng hơn, template tài liệu đã hình thành, website hoạt động ổn định trên Supabase, và quá trình mô phỏng AWS bằng LocalStack – Docker đã sẵn sàng cho các bước triển khai tiếp theo.
