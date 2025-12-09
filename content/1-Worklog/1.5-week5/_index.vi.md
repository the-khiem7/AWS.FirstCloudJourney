---
title : "Worklog Tuần 5"

weight : 5
chapter : false
pre : " <b> 1.5. </b> "
---
## Week 5 Objectives:
- Hiểu rõ **Shared Responsibility Model** và nền tảng bảo mật trên AWS.  
- Thành thạo các dịch vụ Identity: **IAM, Identity Center, AWS Organizations, Cognito**.  
- Nắm vững **Key Management Service (KMS)**, CloudTrail, Athena để phân tích truy vết bảo mật.  
- Thực hành đầy đủ các chiến lược quản trị tài nguyên: tagging, resource groups, restriction policy.  
- Sử dụng IAM Roles, Policies, Access Keys, Switch Role và giới hạn truy cập theo **IP / Time**.  
- Kích hoạt và đánh giá Security Hub để theo dõi posture bảo mật của tài khoản AWS.  

---
## Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | Học **Module 05 – Security Foundation**: <br> + 05-01: Shared Responsibility Model <br> + 05-02: IAM <br> + 05-03: Amazon Cognito <br> + 05-04: AWS Organization <br> + 05-05: AWS Identity Center | 06/10/2025 | 06/10/2025 | [AWS Study Group][1] |
| 2 | Học **Module 05 – Security Services**: <br> + 05-06: Key Management Service (KMS) <br> + 05-07: AWS Security Hub <br> + 05-08: Hands-on & additional research | 07/10/2025 | 07/10/2025 | [AWS Study Group][1] |
| 3 | Thực hành **Security Hub – Lab18**: <br> + Enable Security Hub <br> + Score criteria <br> + Cleanup <br><br> Thực hành **Automation with Tags – Lab22**: <br> + Create VPC, SG, EC2 <br> + Incoming Slack webhook <br> + Lambda Role + Start/Stop Instance Functions <br> + Tag-based automation <br> + Cleanup | 08/10/2025 | 08/10/2025 | [AWS Study Group][1] |
| 4 | Thực hành **Resource Governance – Lab27 + Lab30**: <br> + Create EC2 with tag <br> + Manage tags via console & CLI <br> + Filter resources by tag <br> + Create Resource Group <br> + Create restriction policy <br> + Create IAM limited user & test limits <br> + Cleanup | 09/10/2025 | 09/10/2025 | [AWS Study Group][1] |
| 5 | Thực hành nâng cao **IAM & KMS – Lab28, Lab33, Lab44, Lab48**: <br> + Lab28: IAM user, policy, role, switch role (Tokyo / Virginia), tag-based restriction <br> + Lab33: Create KMS key, CloudTrail logging, Athena query logs, S3 encryption workflow <br> + Lab44: IAM group, admin role, switch role restrictions (IP/Time), cleanup <br> + Lab48: Access keys, IAM role usage with EC2/S3, cleanup | 10/10/2025 | 10/10/2025 | [AWS Study Group][1] |

[1]: https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i

---
## Week 5 Achievements:
- Hiểu rõ kiến trúc bảo mật AWS và cách phân chia trách nhiệm giữa AWS & khách hàng.  
- Thành thạo cách tạo user, policy, group, role, switch role và cấu hình IAM nâng cao.  
- Vận dụng tagging để tự động hóa, phân nhóm, giới hạn truy cập và kiểm soát resource hiệu quả.  
- Kích hoạt Security Hub, đọc security score, và hiểu cách cải thiện posture bảo mật.  
- Sử dụng KMS để mã hóa dữ liệu, kết hợp CloudTrail & Athena để phân tích truy vết.  
- Thiết lập giới hạn truy cập IAM theo **địa chỉ IP** và **khung giờ (time-based condition)**.  
- Hoàn thành toàn bộ workflow: IAM → Security Hub → KMS → CloudTrail → Athena → Encryption → Governance.  
---
