---
title : "Week 6 Worklog"

weight : 6
chapter : false
pre : " <b> 1.6. </b> "
---
## Week 6 Objectives:
- Ôn tập toàn diện các khái niệm về **Database**: RDBMS, NoSQL, OLTP, OLAP, indexing, replication, backup.  
- Hiểu sâu các dịch vụ database của AWS: **RDS, Aurora, Redshift, ElastiCache**.  
- Thực hành triển khai một ứng dụng dùng RDS trong VPC hoàn chỉnh (subnet group, SG, EC2, backup & restore).  
- Làm quen với **AWS Database Migration Service (DMS)** và **Schema Conversion Tool (SCT)**.  
- Triển khai migration từ MSSQL và Oracle sang Aurora MySQL, bao gồm troubleshoot và tối ưu.  
- Làm việc với S3 như một điểm trung gian trong pipeline migration.  

---
## Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | Học **Module 06 – Database Services Overview**: <br> + 06-01: Database Concepts review <br> + 06-02: Amazon RDS & Aurora <br> + 06-03: Redshift & ElastiCache | 13/10/2025 | 13/10/2025 | [AWS Study Group][1] |
| 2 | Thực hành **RDS Deployment – Lab05 (Phần 1)**: <br> + 2.1: Create VPC <br> + 2.2: EC2 Security Group <br> + 2.3: RDS Security Group <br> + 2.4: DB Subnet Group | 14/10/2025 | 14/10/2025 | [AWS Study Group][1] |
| 3 | Thực hành **RDS Deployment – Lab05 (Phần 2)**: <br> + 3: Create EC2 instance <br> + 4: Create RDS instance <br> + 5: Application Deployment <br> + 6: Backup & Restore <br> + 7: Cleanup | 15/10/2025 | 15/10/2025 | [AWS Study Group][1] |
| 4 | Thực hành **Migration Tools – Lab43 (Phần 1)**: <br> + 01: EC2 Connect RDP Client <br> + 02: Fleet Manager <br> + 03: SQL Server source config <br> + 04–05: Oracle source config <br> + 06: Drop constraint <br> + 07–09: MSSQL → Aurora MySQL (target setup + project + schema conversion) | 16/10/2025 | 16/10/2025 | [AWS Study Group][1] |
| 5 | Thực hành **Migration Tools – Lab43 (Phần 2)**: <br> + 10: Oracle → MySQL schema conversion <br> + 11: Create migration task & endpoints <br> + 12: Inspect S3 <br> + 13: Serverless migration <br> + 14: Event notifications <br> + 15: Logs <br> + 16–17: Troubleshoot test scenarios (Memory pressure, table errors) | 17/10/2025 | 17/10/2025 | [AWS Study Group][1] |

[1]: https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i

---
## Week 6 Achievements:
- Củng cố kiến thức về database và hiểu cách AWS tối ưu hóa RDS và Aurora cho hiệu năng & tính sẵn sàng.  
- Tự triển khai full-stack app với backend dùng RDS trong môi trường VPC an toàn.  
- Thực hành backup, restore và hiểu ảnh hưởng của multi-AZ, replication, storage autoscaling.  
- Làm chủ quy trình migration từ MSSQL / Oracle sang Aurora MySQL bằng DMS + SCT.  
- Thiết lập migration endpoints, conversion project, serverless migration, và S3 inspection.  
- Giải quyết các lỗi migration như memory pressure & table errors.  
- Nâng cao hiểu biết về OLTP vs OLAP với Redshift và caching strategy với ElastiCache.  
---
