---
title: "Week 12 Worklog"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Week 12 Objectives
- Fully master AWS simulation with LocalStack Pro.  
- Deploy infrastructure using Terraform on both LocalStack and real AWS.  
- Integrate NextJS into Amplify (local and cloud).  
- Finalize dataframe design and database structure.  
- Deploy the web application to Amplify + CloudFront and validate system behavior.  
- Prepare for monitoring and operations (CloudWatch, SNS).  

---

## Tasks to be carried out this week

| Day | Task | Start Date | Completion Date | Reference |
|-----|------|------------|-----------------|-----------|
| 1 | - Advanced LocalStack training. <br> - Complete LocalStack Pro setup: API Key, Docker, Terraform apply. <br> - Update Terraform to match diagram, enable CloudFront inside Amplify. <br> - Study how to embed NextJS into Amplify (local). <br> - Implement Cognito in local Amplify. <br> - Deploy PostgreSQL to EC2 and upload images to S3. | 24/11/2025 20:00 | 24/11/2025 22:00 | Internal Notes |
| 2 | - Run Terraform. <br> - Explore LocalStack UI-hosting capabilities. <br> - Create a new Amplify project on LocalStack. <br> - LocalStack: separate Terraform for Amplify, upload NextJS, enable CloudFront & S3 integration. <br> - AWS Cloud: create Amplify instance, push project, enable CloudFront & S3 integration. <br> - Learn AWS real deployment flow, error handling, cost concerns. <br> - Study how Solutions Architects design deployments. | 30/11/2025 15:00 | 01/12/2025 01:30 | Internal Notes |
| 3 | - Deploy website to Amplify (AWS real). <br> - Enable CloudFront from Amplify. <br> - Check if Amplify can create a dedicated S3 bucket. <br> - Integrate Cognito into the web. <br> - Design large dataframe and split into 5 tables. <br> - Deploy AWS services using Terraform. <br> - Create AWS cloud services manually for comparison. | 01/12/2025 22:30 | 02/12/2025 03:00 | Internal Notes |
| 4 | - Assignment: CloudWatch, SNS, S3 image storage, event SDK, VPC/Subnet/IGW/EC2/Endpoint creation. | 02/12/2025 | 02/12/2025 | Internal Notes |
| 5 | - Complete remaining project components. <br> - Fix NAN issue in database. <br> - Build charts. <br> - English workshop by Khiêm, Vietnamese workshop by Thống. <br> - Hào uploads Shiny code. <br> - Workshop must include difficulties and improvement suggestions. | 07/12/2025 00:00 | 08/12/2025 00:00 | Internal Notes |

---

## Week 12 Achievements

### 1. Mastered LocalStack Pro and high-level AWS simulation
- LocalStack Pro fully operational with Docker and Terraform.  
- Simulated nearly all AWS services: Amplify, CloudFront, S3, Cognito, EC2, VPC.  
- Greatly reduces AWS cost during development.

### 2. Terraform successfully deployed on local and AWS cloud
- Terraform scripts updated to match architecture diagram.  
- Successfully applied on LocalStack, then on AWS real.  
- Team learned rollback, debugging, and infrastructure best practices.

### 3. NextJS integrated into Amplify (local + cloud)
- NextJS deployed into Amplify using LocalStack.  
- Successfully deployed on AWS Amplify with CloudFront enabled.  
- Represents a complete functioning deployment pipeline.

### 4. Completed dataframe design and database structure
- Large dataframe reorganized into 5 optimized tables.  
- PostgreSQL deployed to EC2; S3 used for product images.  
- Database foundation now stable and production-ready.

### 5. Deployed e-commerce website to AWS Amplify
- Successful Amplify deployment with CloudFront distribution.  
- Verified possibility of creating a dedicated S3 bucket for Amplify.  
- Cognito fully integrated and functioning.

### 6. Strengthened operational and cloud deployment skills
- Team gained hands-on experience deploying Terraform to AWS.  
- Learned real-world cloud deployment techniques similar to Solutions Architects.  
- Improved understanding of cost, security, and troubleshooting.

### 7. Completed workshops and project documentation
- English and Vietnamese workshops completed.  
- Workshops include difficulties, lessons learned, and improvement suggestions.  
- Shiny code uploaded for data visualization.

### 8. Fixed system issues and completed analytical charts
- NAN value issue corrected in database.  
- Visualization charts completed for analysis section.  
- Project now ready for final submission and review.

## Week 12 Summary
Week 12 marks the transition from simulated infrastructure to real AWS deployment.  
The team achieved full deployment: Amplify, CloudFront, S3, Cognito, EC2, Terraform, and LocalStack Pro.  
All documentation, workshops, and data pipelines were finalized, making the project production-ready.
