---
title : "Week 6 Worklog"

weight : 6
chapter : false
pre : " <b> 1.6. </b> "
---
## Week 6 Objectives:
- Review fundamental **database concepts**: RDBMS, NoSQL, OLTP/OLAP, indexing, replication, backup strategies.  
- Deep understanding of AWS database services: **RDS, Aurora, Redshift, ElastiCache**.  
- Deploy a complete application environment using RDS in a secure VPC setup.  
- Learn **AWS Database Migration Service (DMS)** and **Schema Conversion Tool (SCT)** for real-world migrations.  
- Perform migrations from MSSQL and Oracle to Aurora MySQL, including troubleshooting and optimization.  
- Utilize S3 as an intermediate storage layer in migration workflows.  

---
## Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | Study **Module 06 – Database Overview**: <br> + 06-01: Database Concepts review <br> + 06-02: Amazon RDS & Aurora <br> + 06-03: Redshift & ElastiCache | 13/10/2025 | 13/10/2025 | [AWS Study Group][1] |
| 2 | Perform **RDS Deployment – Lab05 (Part 1)**: <br> + 2.1: Create VPC <br> + 2.2: EC2 Security Group <br> + 2.3: RDS Security Group <br> + 2.4: DB Subnet Group | 14/10/2025 | 14/10/2025 | [AWS Study Group][1] |
| 3 | Perform **RDS Deployment – Lab05 (Part 2)**: <br> + 3: Launch EC2 instance <br> + 4: Create RDS instance <br> + 5: Deploy application <br> + 6: Backup & Restore workflow <br> + 7: Cleanup | 15/10/2025 | 15/10/2025 | [AWS Study Group][1] |
| 4 | Perform **Migration Tools – Lab43 (Part 1)**: <br> + 01: EC2 Connect RDP Client <br> + 02: Fleet Manager <br> + 03: Config SQL Server source <br> + 04–05: Config Oracle source DB <br> + 06: Drop constraint <br> + 07–09: MSSQL → Aurora MySQL (target setup + project + schema conversion) | 16/10/2025 | 16/10/2025 | [AWS Study Group][1] |
| 5 | Perform **Migration Tools – Lab43 (Part 2)**: <br> + 10: Oracle → MySQL schema conversion <br> + 11: Create migration task & endpoints <br> + 12: Inspect S3 intermediate files <br> + 13: Serverless migration <br> + 14: Event notifications <br> + 15: Logs <br> + 16–17: Troubleshoot scenarios (Memory pressure, table structure errors) | 17/10/2025 | 17/10/2025 | [AWS Study Group][1] |

[1]: https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i

---
## Week 6 Achievements:
- Rebuilt core understanding of database principles and the rationale behind AWS managed DB services.  
- Successfully deployed a secure VPC-based application using RDS and EC2.  
- Practiced backup & restore workflows and learned Multi-AZ and storage autoscaling implications.  
- Gained hands-on experience using DMS + SCT for MSSQL and Oracle migrations to Aurora MySQL.  
- Configured migration tasks, endpoints, schema conversion, and serverless pipelines.  
- Investigated S3 migration artifacts and resolved migration errors including memory pressure and table incompatibilities.  
- Strengthened knowledge of analytical workloads with Redshift and caching with ElastiCache.  
---
