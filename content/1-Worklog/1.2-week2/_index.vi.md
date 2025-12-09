---
title : "Worklog Tuần 2"

weight : 2
chapter : false
pre : " <b> 1.2. </b> "
---
## Mục tiêu Tuần 2:
- Hiểu nền tảng về **AWS VPC** và các khái niệm mạng cốt lõi trong AWS.  
- Nắm được cơ chế bảo mật VPC: **Security Group**, **Network ACL**, Routing, Subnet, IGW, NAT.  
- Tiếp cận các mô hình hybrid networking: **VPN**, **DirectConnect**, **Route 53 Resolver**.  
- Thực hành toàn bộ VPC labs từ cơ bản đến nâng cao: tạo VPC, subnet, routing, DNS resolver, VPC peering và Transit Gateway.  
- Củng cố kỹ năng thiết kế mạng trong AWS cho các module tiếp theo.

---
## Nhiệm vụ thực hiện trong tuần:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | Học **Module 02 – VPC Core Concepts**: <br> + 02-01: AWS Virtual Private Cloud <br> + 02-02: VPC Security & Multi-VPC Features <br> + 02-03: VPN, DirectConnect, LoadBalancer & Extra Resources | 15/09/2025 | 15/09/2025 | [AWS Study Group][1] |
| 2 | Thực hành **VPC Labs – Lab03 (Phần 1)**: <br> + 02-Lab03-01 → Intro & VPN site-to-site <br> + Subnets: 01.1 <br> + Route Table: 01.2 <br> + Internet Gateway: 01.3 <br> + NAT Gateway: 01.4 | 16/09/2025 | 16/09/2025 | [AWS Study Group][1] |
| 3 | Thực hành **VPC Labs – Lab03 (Phần 2)**: <br> + 02-Lab03-02.1 → Security Group <br> + 02-Lab03-02.2 → Network ACLs <br> + 02-Lab03-02.3 → VPC Resource Map <br> + 02-Lab03-03.x → VPC, Subnet, IGW, Route Table, SG <br> + 02-Lab03-04.x → EC2, Test Connection, NAT Gateway, EC2 Instance Connect Endpoint | 17/09/2025 | 17/09/2025 | [AWS Study Group][1] |
| 4 | Thực hành **Hybrid DNS – Lab10**: <br> + 10-01, 10-02.x, 10-03 <br> + 10-05.x cấu hình DNS <br> + Kiểm thử & cleanup | 18/09/2025 | 18/09/2025 | [AWS Study Group][1] |
| 5 | Thực hành **VPC Peering & Transit Gateway**: <br> + Lab19 → Peering (Intro → ACL → Peering → Route → DNS → Cleanup) <br> + Lab20 → Transit Gateway, attachment, routing | 19/09/2025 | 19/09/2025 | [AWS Study Group][1] |

[1]: https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i

---
## Thành tựu Tuần 2:
- Hiểu rõ cấu trúc mạng AWS và cách thiết kế VPC từ đầu.  
- Áp dụng thực tế **IGW, NAT Gateway, Route Table, Subnet**, và bảo mật mạng.  
- Phân biệt rõ giữa **Security Group (stateful)** và **Network ACL (stateless)**.  
- Làm chủ quy trình thiết lập DNS lai (Hybrid DNS) với Route 53 Resolver.  
- Hoàn thành toàn bộ labs về **VPC Peering** và **Transit Gateway**.  
- Tự tin xây dựng topology mạng quy mô nhỏ đến trung bình phục vụ kiến trúc thực tế.  
---
