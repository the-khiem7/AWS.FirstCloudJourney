---
title : "Week 4 Worklog"

weight : 4
chapter : false
pre : " <b> 1.4. </b> "
---
## Week 4 Objectives:
- Hiểu sâu về **dịch vụ lưu trữ trên AWS**, bao gồm S3, Glacier, Snow Family, Storage Gateway và Backup.  
- Nắm chắc các cơ chế truy cập & bảo mật S3: Access Point, Storage Class, ACL, CORS, Versioning, Replication.  
- Thực hành end-to-end: static website hosting, CloudFront CDN, replication multi-region.  
- Nghiên cứu quy trình migrate VM on-premises lên AWS bằng S3 + VM Import/Export.  
- Thành thạo quản lý file system Multi-AZ, hiệu năng, shadow copy, deduplication, và scaling.

---
## Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | Học **Module 04 – Storage Services Overview**: <br> + 04-01: Dịch vụ lưu trữ trên AWS <br> + 04-02: S3 – Access Point – Storage Class <br> + 04-03: Static Website, CORS, Control Access, Glacier | 29/09/2025 | 29/09/2025 | [AWS Study Group][1] |
| 2 | Học **Module 04-04 – Snow Family, Storage Gateway, Backup** | 30/09/2025 | 30/09/2025 | [AWS Study Group][1] |
| 3 | Thực hành **AWS Backup – Lab13** (v2 theo Module 04): <br> + 02.1: Create S3 Bucket <br> + 02.2: Deploy Infrastructure <br> + 03: Create Backup Plan <br> + 04: Notifications <br> + 05: Test Restore <br> + 06: Cleanup | 01/10/2025 | 01/10/2025 | [AWS Study Group][1] |
| 4 | Thực hành **VM Import/Export – Lab14**: <br> + 01: VMWare Workstation <br> + 02.x: Export VM, Upload to AWS, Import, Deploy Instance from AMI <br> + 03.x: S3 ACL, Export VM, Cleanup | 02/10/2025 | 02/10/2025 | [AWS Study Group][1] |
| 5 | Thực hành **Storage Gateway & File System – Lab24 & Lab25**: <br> + Lab24: Create Storage Gateway, File Shares, Mount on-premises, Cleanup <br> + Lab25: Multi-AZ file system, performance test, deduplication, shadow copies, user quotas, scaling, delete environment <br> + Lab57 (Storage Advanced): S3 static website, CloudFront, public access, versioning, replication multi-region | 03/10/2025 | 03/10/2025 | [AWS Study Group][1] |

[1]: https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i

---
## Week 4 Achievements:
- Thành thạo các tính năng quan trọng của S3: Storage Class, Access Point, ACL, CORS, Performance, Glacier.  
- Thiết lập thành công static website hosting + CloudFront CDN và cấu hình truy cập công khai/an toàn.  
- Thực hành đầy đủ quy trình Backup: lên plan, gán resource, nhận thông báo, test restore, cleanup.  
- Hoàn thiện quy trình **Import/Export VM** từ on-premises sang AWS và triển khai bằng AMI.  
- Thiết lập Storage Gateway & File System: Multi-AZ, performance test, deduplication, snapshot, user quotas.  
- Thực hiện replication S3 multi-region và quản lý versioning hiệu quả.  
- Nâng cao kỹ năng tổng hợp về lưu trữ, bảo mật, hybrid integration và performance tuning trên AWS.  
---
