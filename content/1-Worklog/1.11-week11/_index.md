---
title: "Week 11 Worklog"
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

## Week 11 Objectives
- Continue refining system architecture and deployment flow.  
- Complete the documentation template and verify content quality.  
- Test the e-commerce website thoroughly in practical scenarios.  
- Deepen understanding of Docker and LocalStack in preparation for AWS deployment.  
- Activate GitHub Student benefits and development tools.

---

## Tasks to be carried out this week

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Review docx template. <br> - Run and analyze the project. <br> - Test the stability of the e-commerce website. <br> - Use LocalStack to test AWS services. <br> - Activate GitHub Student. <br> - Learn Docker and LocalStack. | 21/11/2025 12:00 | 21/11/2025 14:00 | Internal Notes |
| 2 | - Test the e-commerce site thoroughly. <br> - Validate product view, cart, and checkout flows. <br> - Attempt Clerk authentication → failed → switch to Cognito. <br> - Deploy web to Vercel (failed 4 times, succeeded on the 5th). <br> - Fixed next.config.ts, package.json, duplicate configs, eslint issues. <br> - Added .env file. <br> - Successfully stored Cognito userId into DB. | 23/11/2025 23:00 | 24/11/2025 01:00 | Internal Notes |

---

## Week 11 Achievements

### 1. Improved documentation template quality
- The docx template was reviewed and refined for clarity, structure, and consistency.  
- Updated formatting ensures better readability and professional presentation.  
- Forms the basis for final workshop documents and Week 12 reporting.

### 2. Fully tested the e-commerce website
- Real usage scenarios were simulated: product browsing, cart handling, and checkout.  
- Helped uncover UI/UX issues and validate core business logic.  
- This is a critical milestone before integrating AWS services.

### 3. Migrated authentication from Clerk to Cognito
- Clerk authentication could not return userId to the database, causing a system mismatch.  
- Team switched to AWS Cognito, which aligns with the cloud architecture.  
- Provides unified authentication for upcoming AWS integrations.

### 4. Successfully deployed the website to Vercel
- After multiple failed attempts, deployment succeeded.  
- Fixed configuration issues involving next.config.ts, eslint, and duplicated configs.  
- The site is now publicly accessible for testing and refinement.

### 5. UserId successfully stored in database
- Cognito userId is now stored in Supabase/PostgreSQL.  
- This enables advanced features such as order tracking and behavioral analytics.

### 6. Mastered Docker and LocalStack essentials
- Docker now used to standardize local development environments.  
- LocalStack provides a near-complete AWS simulation for safe testing.  
- Essential preparation for deploying infrastructure on real AWS.

### 7. GitHub Student activated
- Provides valuable cloud and development credits.  
- Enables experimentation and scaling without financial concerns.

## Week 11 Summary
Week 11 strengthened development tools, validated the website in real scenarios, stabilized authentication, and refined documentation.  
The team is now fully prepared to begin real AWS deployment and infrastructure work in Week 12.
