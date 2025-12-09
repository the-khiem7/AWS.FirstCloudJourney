---
title : "Worklog Tuần 8"
weight : 8
chapter : false
pre : " <b> 1.8. </b> "
---

## Mục tiêu Tuần 8
- Tổ chức buổi gặp mặt team đầu tiên để thống nhất quy trình, vai trò và hướng đi dự án.  
- Thiết lập hạ tầng quản lý mã nguồn và tài liệu (GitHub, GitHub Pages).  
- Bắt đầu nghiên cứu các dịch vụ AWS cốt lõi phục vụ dự án thương mại điện tử: Cognito, S3.  
- Chuẩn hóa proposal và định hướng roadmap đến tuần 12.  
- Xác định nền tảng kiến trúc kỹ thuật: dùng hay không dùng backend riêng, chọn ORM, cân nhắc database, định hướng cho Clickstream.  

---

## Các công việc cần thực hiện trong tuần

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Gặp mặt team trực tiếp. <br> - Leader upload nội dung dự án lên GitHub. <br> - Thêm thành viên vào repository. <br> - Kiểm tra, sửa lỗi và chuẩn hóa proposal. <br> - Chốt nhiệm vụ nghiên cứu Cognito, S3 và các dịch vụ AWS khác, thống nhất mục tiêu hoàn thành trước tuần 12. | 28/10/2025 09:00 | 28/10/2025 12:00 | Internal Notes |
| 2 | - Nghiên cứu Cognito và S3. <br> - Bắt đầu build web thương mại điện tử từ template có sẵn. <br> - Tích hợp dịch vụ AWS vào web. <br> - Thảo luận và thống nhất không dùng backend riêng. <br> - Tìm hiểu Clickstream: nguyên lý, thuật toán, code base, quy trình tích hợp. <br> - Xác định phương án database. <br> - Chỉnh sửa lại kiến trúc hệ thống. <br> - Nghiên cứu tổng quan ORM của template frontend. | 29/10/2025 22:30 | 30/10/2025 00:30 | Internal Notes |

---

## Thành tựu Tuần 8

### 1. Củng cố tổ chức và quy trình làm việc của team
- Sau hơn một tháng hoạt động online, team đã có buổi gặp mặt trực tiếp đầu tiên.  
- Cuộc họp giúp làm rõ vai trò từng thành viên, cách phối hợp, cách nhận và báo cáo nhiệm vụ.  
- Sau buổi gặp mặt, tinh thần làm việc của team gắn kết hơn, cam kết hơn với mục tiêu hoàn thành trước tuần 12.

### 2. Thiết lập nền tảng quản lý dự án trên GitHub
- Repository chính thức được thiết lập, là nơi chứa toàn bộ tài liệu và mã nguồn.  
- Proposal được rà soát kỹ lưỡng, sửa lỗi nội dung và định dạng để dễ phát triển và theo dõi.  
- GitHub Pages được chuẩn bị để public tài liệu hoặc làm landing page mô tả dự án.  
- Từ thời điểm này, dự án có "single source of truth", giảm phân mảnh thông tin.

### 3. Hiểu nền tảng Cognito và S3 để xây dựng kiến trúc serverless
- Team đã nắm được cơ chế xác thực của Cognito, cách hoạt động của user pool và token.  
- Tìm hiểu cách S3 lưu trữ asset, hình ảnh và tài nguyên tĩnh cho web.  
- Nền tảng kiến thức này hỗ trợ quyết định xây dựng hệ thống không cần backend truyền thống.

### 4. Đưa ra quyết định quan trọng: không xây dựng backend riêng
- Qua phân tích workload và mục tiêu thời gian, team quyết định không tự dựng backend bằng Node.js hoặc Python.  
- Thay vào đó, hệ thống sẽ dựa trên AWS để xử lý các tác vụ cần thiết (Cognito, S3, sau này có thể thêm Lambda hoặc database).  
- Giúp giảm chi phí phát triển, tránh phức tạp không cần thiết và tập trung vào các phần cốt lõi.

### 5. Bắt đầu xây dựng web thương mại điện tử
- Sử dụng template frontend có sẵn, giúp rút ngắn thời gian thiết kế UI.  
- Hệ thống cơ bản đã hình thành: xem sản phẩm, giỏ hàng, luồng mua hàng, đăng nhập.  
- Đánh dấu sự chuyển dịch từ giai đoạn chuẩn bị sang giai đoạn xây dựng sản phẩm thực tế.

### 6. Phân tích chuyên sâu Clickstream
- Team nghiên cứu nguyên lý tracking event, các loại dữ liệu thu thập và dòng chảy dữ liệu.  
- Tìm hiểu mô hình pipeline phân tích hành vi người dùng.  
- Dù khó, nhưng nghiên cứu sớm tạo nền tảng tốt cho phân tích dữ liệu sau này.

### 7. Hoàn thiện phiên bản đầu tiên của kiến trúc hệ thống
- Kiến trúc hệ thống được cập nhật phản ánh quyết định sử dụng Cognito + S3 và bỏ backend riêng.  
- Đây là phiên bản kiến trúc nền, giúp tất cả thành viên có chung nhận thức về mô hình hệ thống.  
- Sẵn sàng để cải thiện qua các tuần tiếp theo.

### 8. Nắm bắt cơ chế ORM trong template
- Nhận diện cách ORM định nghĩa model, xử lý truy xuất dữ liệu và tổ chức thư mục.  
- Hỗ trợ chuẩn bị cho việc tích hợp database thật như Supabase, PostgreSQL hoặc RDS.  
- Giảm rủi ro viết lại code và giúp code base sạch hơn khi mở rộng.

## Tổng kết Tuần 8
Tuần 8 là tuần bản lề, chuyển từ khâu chuẩn bị sang xây dựng thực tế.  
Team đã thống nhất mô hình làm việc, thiết lập công cụ quản lý trung tâm, nghiên cứu các dịch vụ AWS quan trọng và bắt đầu phát triển web.  

Nền tảng từ tuần này giúp các tuần tiếp theo tập trung mạnh vào xây dựng tính năng và tích hợp cloud.
