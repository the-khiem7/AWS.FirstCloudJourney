---
title : "Week 5 Worklog"

weight : 5
chapter : false
pre : " <b> 1.5. </b> "
---
## Week 5 Objectives:
- Understand the **Shared Responsibility Model** and AWS security fundamentals.  
- Gain proficiency in identity services: **IAM, AWS Identity Center, AWS Organizations, Cognito**.  
- Learn and apply **KMS**, CloudTrail, and Athena for audit logging and security analytics.  
- Practice governance strategies: tagging, resource groups, restriction policies, automation.  
- Manage IAM Roles, Policies, Access Keys, Switch Roles, and conditional access based on **IP / Time**.  
- Enable and evaluate **Security Hub** for security posture assessment.  

---
## Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | Study **Module 05 – Security Fundamentals**: <br> + 05-01: Shared Responsibility Model <br> + 05-02: IAM <br> + 05-03: Cognito <br> + 05-04: AWS Organization <br> + 05-05: Identity Center | 06/10/2025 | 06/10/2025 | [AWS Study Group][1] |
| 2 | Study **Module 05 Security Services**: <br> + 05-06: Key Management Service <br> + 05-07: Security Hub <br> + 05-08: Hands-on & research | 07/10/2025 | 07/10/2025 | [AWS Study Group][1] |
| 3 | Perform **Security Hub – Lab18**: <br> + Enable Security Hub <br> + Evaluate security scores <br> + Cleanup <br><br> Perform **Tag Automation – Lab22**: <br> + Create VPC, SG, EC2 <br> + Slack webhook <br> + Lambda role + Start/Stop EC2 Functions <br> + Tag-based automation <br> + Cleanup | 08/10/2025 | 08/10/2025 | [AWS Study Group][1] |
| 4 | Perform **Resource Governance – Lab27 + Lab30**: <br> + Create EC2 with tags <br> + Manage tags via console & CLI <br> + Filter resources by tag <br> + Create Resource Group <br> + Create restriction policy <br> + Create limited IAM user & test limits <br> + Cleanup | 09/10/2025 | 09/10/2025 | [AWS Study Group][1] |
| 5 | Advanced IAM & KMS Practice – **Lab28, Lab33, Lab44, Lab48**: <br> + Lab28: IAM user, policy, role, switch role (Tokyo/Virginia), tag-based restrictions <br> + Lab33: Create KMS key, CloudTrail logging, Athena for log analysis, S3 encryption <br> + Lab44: IAM groups, admin role, IP/Time-based Switch Role restrictions, cleanup <br> + Lab48: Access keys, IAM role usage with EC2/S3, cleanup | 10/10/2025 | 10/10/2025 | [AWS Study Group][1] |

[1]: https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i

---
## Week 5 Achievements:
- Mastered AWS security fundamentals and the Shared Responsibility Model.  
- Learned advanced IAM: user, groups, roles, policies, Switch Role, conditional access.  
- Applied tagging for governance, automation, filtering, and resource grouping.  
- Enabled Security Hub, evaluated findings, and understood improvement strategies.  
- Used KMS for encryption, CloudTrail for audit logs, Athena for querying security insights.  
- Implemented IP- and Time-based access controls using IAM conditions.  
- Completed an end-to-end workflow integrating IAM → Security Hub → KMS → CloudTrail → Athena → S3 Encryption → Governance.  
---
