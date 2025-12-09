---
title: "Week 9 Worklog"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

## Week 9 Objectives
- Finalize the build of the e-commerce web application and start loading initial datasets.  
- Clarify the role of CloudFront and determine where AWS Amplify fits in the architecture.  
- Set up a local development environment using LocalStack to simulate AWS services.  
- Standardize the Python library stack, versions, and packaging approach.  
- Continue researching ORM and Clickstream to prepare for user behavior analysis.

---

## Tasks to be carried out this week

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Completed the web build. <br> - Began preparing sample data. <br> - Explored expanded use cases of CloudFront. <br> - Clarified CloudFront–Amplify sequence in deployment. <br> - Set up LocalStack for offline AWS simulation. <br> - Finalized Python libraries and versions. <br> - Continued ORM development and Clickstream research (until 07/11). | 05/11/2025 14:15 | 05/11/2025 15:40 | Internal Notes |
| 2 | - Reviewed CloudFront usage and optimization. <br> - Chose LocalStack BASE to support Cognito. <br> - Updated website logic and migrated database to Supabase. <br> - Adjusted CloudFront–Amplify architecture flow. <br> - Updated architecture with Amplify and PostgreSQL. <br> - Finalized Python dependency stack. <br> - Updated proposal alignment with new project goals. | 09/11/2025 20:30 | 09/11/2025 00:00 | Internal Notes |

---

## Week 9 Achievements

### 1. Completed web build and started handling data
- The e-commerce web application was successfully built.  
- Began loading real sample data to validate product, cart, and user flows.  
- Transitioned from initialization to real functional testing.

### 2. Deep understanding of CloudFront in the architecture
- CloudFront is now fully understood beyond speed improvement: security, caching, origin shielding, edge distribution.  
- Team confirmed that Amplify should sit behind CloudFront in the deployment flow.  
- This clarity ensures correct architectural design moving forward.

### 3. LocalStack established as the main local AWS simulation tool
- LocalStack BASE was selected to ensure Cognito compatibility.  
- Helps test AWS infrastructure without incurring cloud costs.  
- Speeds up development and reduces deployment risks.

### 4. Standardized Python environment
- Unified Python versions and libraries across team members.  
- Prevents inconsistency and dependency conflicts.  
- Prepares the foundation for automation, Clickstream handling, and packaging.

### 5. Architecture adjustments for better clarity and scalability
- Amplify and PostgreSQL were added into the main architecture.  
- The updated flow better reflects how the real AWS deployment will operate.  
- Improved alignment across team members.

### 6. Refined website and integrated Supabase database
- The database layer was migrated to Supabase for faster development.  
- Frontend integrates cleanly with the new backend structure.  
- Simplifies early-stage development and reduces setup time.

### 7. Continued research on Clickstream and ORM
- Team explored how event tracking works and how pipelines process user behavior data.  
- ORM understanding improved, preparing the team for future integration with AWS or cloud databases.  
- Important groundwork for analytical features.

## Week 9 Summary
Week 9 strengthened the architectural clarity, completed the initial web build, aligned the environment setup, and prepared the team for upcoming AWS integration.  
These achievements build momentum for deeper cloud deployment in Weeks 10–12.
