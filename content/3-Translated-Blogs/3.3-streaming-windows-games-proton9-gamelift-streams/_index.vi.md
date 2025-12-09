---
title: "Phat truc tuyen tro choi Windows bang Proton 9 tren Amazon GameLift Streams"
date: 2025-08-07
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

*Tác giả: Benjamin Meyer, Adam Chernick và Natasha Rooney | ngày 07 tháng 8 năm 2025 | Trong: [Amazon EC2](https://aws.amazon.com/ec2/), [Amazon GameLift](https://aws.amazon.com/gamelift/), [AWS Management Console](https://aws.amazon.com/console/), [Compute](https://aws.amazon.com/blogs/gametech/category/compute/), [Game Development](https://aws.amazon.com/blogs/gametech/category/game-development/), [Games](https://aws.amazon.com/blogs/gametech/category/industries/games/), [Industries](https://aws.amazon.com/blogs/gametech/category/industries/), [Management Tools](https://aws.amazon.com/blogs/gametech/category/management-tools/)*

Người chơi hiện nay mong đợi những trải nghiệm có thể chơi ngay lập tức chỉ bằng một cú nhấp chuột trên mọi thiết bị, và Amazon GameLift Streams giúp khách hàng xây dựng những trải nghiệm như vậy cho cả trò chơi và các ứng dụng streaming.  
 Truyền thống, việc chơi game phụ thuộc nhiều vào hệ điều hành Windows, tuy nhiên Amazon GameLift Streams có thể chạy các trò chơi hoặc ứng dụng được thiết kế cho Windows trong môi trường Windows runtime hoặc Proton runtime.

[Proton](https://github.com/ValveSoftware/Proton) là một lớp tương thích (compatibility layer) cho phép các tệp thực thi của Windows có thể chạy trên hệ điều hành Linux. Proton đạt được điều này bằng cách dịch các lời gọi API của Windows – bao gồm đồ họa và nhập liệu – sang các lệnh tương đương trong Linux, cho phép các nhà phát triển tận dụng hạ tầng máy chủ Linux cho các khối lượng công việc vốn chỉ có thể chạy trên Windows trước đây.

Với cách tiếp cận này, bạn có thể giảm chi phí hạ tầng lên tới 63% khi vận hành một trò chơi ở cùng mức tenancy mà vẫn duy trì hiệu năng tương đương so với môi trường Windows runtime. Linux cũng cho phép sử dụng multi-tenancy, tức có thể chạy hai luồng stream trên một GPU duy nhất, giúp giảm chi phí hơn nữa.

Chúng tôi công bố việc hỗ trợ Proton 9 như một tùy chọn runtime mới cho Amazon GameLift Streams, cải thiện hơn nữa khả năng tương thích với các bản build Windows trước đây. Trong bài viết này, chúng tôi sẽ tìm hiểu sâu hơn về cách Proton 9 hoạt động và cách bạn có thể sử dụng nó để giảm chi phí stream các trò chơi hoặc ứng dụng Windows tới người dùng của mình.

---

## **Các khái niệm về Amazon GameLift Streams**

Amazon GameLift Streams cho phép bạn stream trò chơi với độ phân giải lên đến 1080p và 60 khung hình mỗi giây tới bất kỳ thiết bị nào có trình duyệt.  
 Nhờ tận dụng hạ tầng toàn cầu của AWS và các GPU instance, bạn có thể triển khai và stream nội dung trò chơi trong vòng vài phút mà không cần sửa đổi mã nguồn, và người chơi có thể bắt đầu chơi chỉ trong vài giây mà không cần cài đặt.

Amazon GameLift Streams sử dụng một số thuật ngữ chính để giúp bạn thiết lập và quản lý luồng stream nhanh hơn. Hiểu rõ các khái niệm này sẽ giúp bạn cấu hình các tùy chọn phù hợp:

* **Môi trường runtime (Runtime environment):** Là hệ điều hành và phần mềm nền chạy trên các instance Amazon Elastic Compute Cloud ([Amazon EC2](https://aws.amazon.com/ec2/)) – nơi vận hành ứng dụng của bạn. Các tùy chọn bao gồm: Windows, Ubuntu và Proton.

* **Lớp stream (Stream class):** Xác định loại instance EC2 và số lượng luồng stream có thể chạy đồng thời trên một instance.

* **Nhóm stream (Stream group):** Một cấu trúc bao gồm ứng dụng của bạn, lớp stream và dung lượng stream.

* **Dung lượng stream (Stream capacity):** Một giá trị số thể hiện số phiên stream mà nhóm stream có thể chạy đồng thời.

---

## **Tìm hiểu về Proton**

Proton là một lớp tương thích mã nguồn mở do Valve phát triển phối hợp cùng CodeWeavers, cho phép trò chơi Windows chạy trên hệ thống Linux.  
 Nền tảng của Proton dựa trên dự án Wine (Wine Is Not an Emulator) – cung cấp khả năng dịch các lời gọi API của Windows. Tuy nhiên, Wine có thể gặp khó khăn khi xử lý các đồ họa 3D hiện đại, khiến việc sử dụng nó cho các trò chơi 3D hiện đại trở nên thiếu ổn định.

Proton mở rộng năng lực của Wine bằng cách bổ sung một lớp dịch nâng cao DirectX-to-Vulkan (DXVK).  
 DXVK có nhiệm vụ dịch các lời gọi API đồ họa đặc thù của Windows (DirectX) sang các API đồ họa chạy trên Linux (Vulkan). DXVK hỗ trợ DirectX 9, 10 và 11\.  
 Đối với DirectX 12, Proton bổ sung thêm vkd3d-Proton làm cầu dịch thứ hai.  
 Với hệ thống này, Proton có thể chuyển đổi toàn bộ các lời gọi API đồ họa và phi đồ họa của Windows sang các thao tác tương đương trên Linux và Vulkan.

[![Diagram showing how a Windows game uses Wine, DXVK, or vkd3d-Proton to translate Windows system and DirectX graphics calls for execution on Linux. DXVK handles DirectX 9–11, vkd3d handles DirectX 12, both targeting the Linux GPU driver.](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2025/08/06/Proton-Image-1.png)](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2025/08/06/Proton-Image-1.png)

*Figure 1: A diagram of which system calls which translation layer.*

Amazon GameLift Streams đóng gói các phiên bản Proton được tùy chỉnh: 8.0-2c, 8.0-5, và hiện nay là 9.0-2 (2024) — tất cả đều được tối ưu hóa đặc biệt cho hạ tầng và công nghệ stream của AWS.

---

## **Lợi ích kỹ thuật và kinh tế**

Lợi ích của việc sử dụng runtime Proton 9 trong Amazon GameLift Streams so với Windows runtime không chỉ dừng lại ở việc tiết kiệm chi phí bản quyền.  
 Các lớp stream dựa trên Windows (ví dụ gen5n\_win2022) phải chịu chi phí giấy phép Windows Server 2022 và hoạt động ở chế độ single-tenant, nghĩa là mỗi máy chỉ có thể phục vụ một phiên stream.

Ngược lại, các stream group sử dụng lớp stream dựa trên Linux có thể vận hành ở chế độ multi-tenant; ví dụ, gen5n\_high có thể phục vụ hai phiên stream từ một GPU duy nhất, đồng thời loại bỏ chi phí bản quyền.  
 Các instance Amazon EC2 cơ sở vẫn thuộc quyền sở hữu riêng của bạn, ngay cả khi nhiều phiên stream cùng chạy trên đó.

Đối với các trò chơi hoặc ứng dụng không yêu cầu hiệu năng cao, Proton 9 là một công cụ quan trọng để tối ưu tổng chi phí hạ tầng.

[![Comparison of cost for each stream hour of stream classes in Ohio.](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2025/08/06/Proton-Image-2.png)](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2025/08/06/Proton-Image-2.png)  

## **Kiểm thử khả năng tương thích trò chơi với Proton 9**

Tuy nhiên, việc chuyển sang Proton 9 không phải là không có các yếu tố kỹ thuật cần xem xét.  
 Một số phần mềm chống gian lận (anti-cheat) yêu cầu cấu hình tương thích với Linux hoặc cần một bản build riêng không có Anti-Cheat.  
 Bất kỳ hoạt động nào ở cấp kernel đều có thể cần cấu hình đặc biệt.  
 Ngoài ra, trò chơi của bạn có thể sử dụng các thao tác mà Proton không thể dịch chính xác hoàn toàn sang API đồ họa Vulkan.

Nếu bạn gặp vấn đề về khả năng tương thích khi sử dụng Proton runtime, hãy thử nghiệm trước với phiên bản Proton 9 mới nhất có sẵn trong Amazon GameLift Streams.  
 Nếu vẫn gặp sự cố, AWS cung cấp [hướng dẫn chi tiết](https://docs.aws.amazon.com/) về cách thiết lập một môi trường Amazon EC2 tương đương để kiểm thử và gỡ lỗi các vấn đề tương thích của Proton.

---

## **Các phương pháp tối ưu chi phí bổ sung**

Việc sử dụng **Proton 9** là một cách tuyệt vời để **chia sẻ GPU** và **loại bỏ chi phí bản quyền Windows**, giúp bạn lựa chọn lớp stream phù hợp nhất cho nhu cầu trò chơi của mình.  
 Các tính năng tối ưu chi phí đóng vai trò quan trọng đối với các trường hợp hướng đến người tiêu dùng, và **Amazon GameLift Streams** cung cấp thêm nhiều chức năng giúp khách hàng quản lý chi phí hiệu quả hơn cho các trường hợp sử dụng streaming.

Điều quan trọng là bạn cần **hiểu rõ dung lượng stream (stream capacity)**.  
 Stream capacity là chỉ số thể hiện số lượng stream có thể được khởi tạo ngay lập tức – có thể được đặt thông qua **API call** hoặc trong **AWS Management Console**.  
 Việc đặt dung lượng sát với nhu cầu thực tế giúp bạn tránh việc trả phí cho công suất dư thừa.  
 Bạn có thể tăng hoặc giảm dung lượng này **thủ công** hoặc **tự động** bằng cách gọi **Amazon GameLift Streams API** dựa trên **sự kiện** (như khi số lượng người chơi tăng lên) hoặc dựa trên **khoảng thời gian** (ví dụ cùng một giờ mỗi ngày).

Đối với các tình huống mà người dùng có thể chờ **tối đa 5 phút** để stream được khởi động (thường là các trường hợp sử dụng nội bộ trong doanh nghiệp), khách hàng có thể tận dụng **on-demand stream capacity**.  
 Ở chế độ này, dung lượng stream chỉ được khởi tạo khi có **yêu cầu phát** và sẽ được **hủy cấp phát** khi phiên stream kết thúc.  
 Điều này đồng nghĩa với việc khách hàng **chỉ trả tiền** cho dung lượng stream **khi có yêu cầu** và **chỉ trong thời gian phiên stream diễn ra**.

---

## **Kết luận**

Runtime **Proton 9** mới mang đến **khả năng tương thích rộng hơn với DirectX 12** và khắc phục các vấn đề đã tồn tại trong những phiên bản trước.  
 Kết hợp với các **lớp stream mới nhất của Amazon GameLift Streams**, bạn có thể **tối ưu hóa chi phí** khi stream các trò chơi AAA và **giảm thêm** chi phí nhờ khả năng **multi-tenancy**, nếu yêu cầu hiệu năng của trò chơi cho phép.

Liên hệ với [**đại diện AWS**](https://pages.awscloud.com/) để tìm hiểu thêm về cách chúng tôi có thể giúp doanh nghiệp của bạn tăng tốc và mở rộng quy mô nhanh hơn.

[![Benjamin Meyer](https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2021/03/30/bemeyer.png)](https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2021/03/30/bemeyer.png)

## **Benjamin Meyer**

**Benjamin Meyer** là **Kiến trúc sư Giải pháp cấp cao (Principal Solutions Architect)** tại **AWS**, tập trung vào các doanh nghiệp thuộc lĩnh vực **Dịch vụ Tài chính** và **Ngành Công nghiệp Trò chơi (Games)** tại khu vực Trung và Đông Âu, giúp họ giải quyết các thách thức kinh doanh thông qua các dịch vụ đám mây của AWS.

---

[![Adam Chernick](https://d2908q01vomqb2.cloudfront.net/a17554a0d2b15a664c0e73900184544f19e70227/2023/08/09/adam-headshot.png)](https://d2908q01vomqb2.cloudfront.net/a17554a0d2b15a664c0e73900184544f19e70227/2023/08/09/adam-headshot.png)

## **Adam Chernick**

**Adam Chernick** là **Kiến trúc sư Giải pháp cấp cao toàn cầu (Worldwide Sr. Solutions Architect)** chuyên trách **Amazon GameLift Streams**.  
 Anh đã dành sự nghiệp của mình cho các lĩnh vực **đồ họa 3D thời gian thực (real-time 3D)**, **AI tạo sinh (generative AI)** và các công nghệ tiên tiến liên quan.

---

[![Natasha Rooney](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2025/06/11/Natasha-Rooney.jpeg)](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2025/06/11/Natasha-Rooney.jpeg)

## **Natasha Rooney**

**Natasha Rooney** là **Kiến trúc sư Giải pháp cấp cao (Senior Solutions Architect)** tại **AWS**, làm việc với các khách hàng trong lĩnh vực **Truyền thông & Giải trí (Media & Entertainment)** và **Trò chơi (Gaming)**, hỗ trợ họ trong việc thiết kế **kiến trúc ứng dụng và hạ tầng** trên nền tảng AWS.  
 Là một **game thủ đam mê**, cô chuyên hỗ trợ các **studio trò chơi** và **đối tác** tối ưu hóa khối lượng công việc (workload) của họ trên AWS.