---
title : "Week 8 Worklog"
weight : 8
chapter : false
pre : " <b> 1.8. </b> "
---
## Week 8 Objectives:
- Run the first in-person team meeting to align workflow, roles, and project direction.  
- Set up project infrastructure on GitHub and GitHub Pages.  
- Start researching core AWS services for the E-commerce project: Cognito, S3, serverless web architecture.  
- Refine and standardize the proposal to match the target of finishing before Week 12.  
- Lay the technical foundation: backend vs. no-backend, ORM, database choice, and Clickstream direction.  

---
## Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|------|------------|-----------------|-------------------|
| 1 | - First offline team meeting. <br> - Leader pushed initial project content to GitHub. <br> - Added all members to the repository. <br> - Reviewed and cleaned up the proposal. <br> - Finalized the research plan for Cognito, S3, other AWS services, with a clear Week 12 deadline. | 28/10/2025 09:00 | 28/10/2025 12:00 | Internal meeting notes |
| 2 | - Researched Amazon Cognito and S3. <br> - Started building the E-commerce web using a Git template. <br> - Integrated AWS services into the web. <br> - Discussed architecture direction and decided not to build a custom backend. <br> - Studied Clickstream in detail (mechanism, algorithms, code, integration flow). <br> - Considered database choices and whether to use AWS-native databases. <br> - Updated the architecture draft. <br> - Explored the ORM embedded in the frontend template. | 29/10/2025 22:30 | 30/10/2025 00:30 | Internal meeting notes |

---
## Week 8 Achievements:

### 1. Solid team alignment and clarified working model
- After more than a month working only online, the team had its first in-person meeting.  
- This meeting clarified roles, task ownership, reporting style, and the shared commitment to finish the project by Week 12.  
- It marks the transition from “loose collaboration” to a more structured, project-like working mode.

### 2. Project infrastructure set up on GitHub and GitHub Pages
- The GitHub repo became the central place for code and documentation.  
- The proposal was reviewed for content consistency and quality.  
- GitHub Pages was prepared as a potential public-facing site for documentation or project overview.  
- From Week 8 onward, the team has a single, authoritative source of project truth.

### 3. Foundation in Cognito and S3 for a serverless web approach
- The team gained working knowledge of Cognito user pools, authentication flows and token handling.  
- S3 was studied as the primary place for static content and product images.  
- This learning is the technical basis of the “no dedicated backend” decision and supports a scalable, serverless web model.

### 4. Key architectural decision: no dedicated backend service
- Instead of building an extra backend (Node.js/Python), the team decided to rely on AWS services (Cognito, S3, potential API Gateway/Lambda, or managed DB like Supabase/PostgreSQL).  
- This significantly reduces complexity and development time, while still aligning well with cloud-native best practices.  
- It frees capacity for focusing on integration, observability and analytics rather than reinventing a traditional backend.

### 5. E-commerce web development bootstrapped
- The team selected an existing UI template, which accelerates the front-end work.  
- Initial structure for the main flows (view products, cart, purchase, login, etc.) is in place.  
- The project officially moves from “design and planning” to “building a real, working application”.

### 6. Initial deep dive into Clickstream tracking
- The team investigated how Clickstream-style tracking works end to end:  
  - How events are captured on the frontend  
  - What kind of data is sent and why  
  - How this data can be integrated into a pipeline for later analysis  
- Although identified as difficult, doing this early gives the team a head start for future analytics features.

### 7. Drafted the first version of the system architecture
- The architecture diagram was updated to reflect the latest decisions: no standalone backend, Cognito and S3 as core components, and an E-commerce-centric design.  
- Even though it is just v1, having a concrete picture helps everyone reason about the system in the same way.  
- It also provides a baseline to iterate on in the following weeks.

### 8. ORM behavior and structure understood
- By reading and understanding the ORM layer in the template, the team reduced future risk around database integration.  
- It will be easier later to connect to a real database (PostgreSQL/Supabase/AWS RDS) without rewriting large parts of the data access code.  
- This is a long-term productivity gain, established early.

## Week 8 Summary:
Week 8 is the week where the project truly “starts moving”:  
- Team structure and workflow are clarified.  
- GitHub/GitHub Pages is set up as the central project hub.  
- Key technical decisions are made, especially around serverless architecture and no backend.  
- The web app is actually being built.  
- Research into Clickstream and ORM prepares the team for upcoming advanced features.  

The foundation laid in Week 8 enables the following weeks to be more execution-focused and technically coherent.
---
