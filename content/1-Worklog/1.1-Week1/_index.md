---
title: "Week 1 Worklog"
date: 2026-06-08
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Week 1 - Program Orientation, System Analysis, and AWS Fundamentals

### Week 1 Objectives:

- Understand the internship requirements, workshop structure, and FCAJ project reporting guidelines.
- Analyze the recruitment and internship application management problem.
- Identify the main user groups, core features, and high-level system architecture.
- Become familiar with AWS Account, AWS Budgets, and access management using AWS Identity and Access Management.
- Initialize the backend foundation and the initial authentication flow.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Reviewed the internship program requirements, the sample workshop structure, and the FCAJ project reporting guidelines. | 08/06/2026 | 08/06/2026 | [FCAJ Project Requirements](https://cloudjourney.awsstudygroup.com/8-fcjworkforce/); [FCAJ Internship Report Sample](https://workshop-sample.awsfcaj.com) |
| 2 | Analyzed the recruitment problem, identified Candidate and HR/Company as the two main user groups, and outlined the initial flows for job posting, job searching, application submission, and candidate management. | 09/06/2026 | 10/06/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html); [FCAJ Project Requirements](https://cloudjourney.awsstudygroup.com/8-fcjworkforce/) |
| 3 | Created and reviewed the AWS Account, studied cost monitoring with AWS Budgets, and learned the principle of least privilege in AWS IAM. | 11/06/2026 | 11/06/2026 | [Creating an AWS Account](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html); [Managing Costs with AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html); [Introduction to AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) |
| 4 | Initialized the FastAPI backend, configured the database connection, created the initial models, prepared Alembic migrations, and implemented registration, login, password hashing, and JWT authentication. | 12/06/2026 | 12/06/2026 | [FastAPI OAuth2 with JWT](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/); [SQLAlchemy ORM Quick Start](https://docs.sqlalchemy.org/en/20/orm/quickstart.html); [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html) |

### Technical Understanding:

During the first week, I focused on understanding the project from both the business and technical perspectives before beginning the AWS deployment activities.

I reviewed the internship requirements, the FCAJ workshop structure, and the reporting guidelines to understand the expected project deliverables. I also analyzed the recruitment and internship application management process and identified two main user groups: Candidates and HR/Company users.

Candidates need to search for jobs, submit applications, upload documents, communicate with recruiters, and receive suitable job recommendations. HR/Company users need to create job postings, manage applications, review candidate profiles, communicate with applicants, and evaluate candidate-job suitability.

From the technical perspective, I studied the initial system structure and identified the main application components:

- React and Vite frontend
- FastAPI backend
- Node.js and Socket.IO chat service
- AI processing service
- PostgreSQL relational database
- Redis for caching and realtime message distribution
- DynamoDB for chat-related data
- Document storage

I also reviewed the backend authentication workflow, including user registration, password hashing, login validation, JWT access token generation, protected API endpoints, SQLAlchemy models, and Alembic database migrations.

### AWS Knowledge Gained:

During Week 1, I studied several AWS fundamentals that would be required for the later deployment phases.

I learned how an AWS Account is structured and why the root account should only be used for account-level administrative tasks. Normal development and deployment activities should instead use IAM users or roles with appropriate permissions.

I studied AWS Identity and Access Management, including IAM users, groups, roles, policies, permissions, and the principle of least privilege. This principle requires each user or service to receive only the permissions necessary to perform its assigned responsibilities.

I also reviewed AWS Budgets and basic cost monitoring. Configuring budget alerts before creating AWS resources is important because services may continue generating charges even when they are not being actively used.

In addition, I reviewed the AWS Well-Architected Framework and its six pillars:

- Operational Excellence
- Security
- Reliability
- Performance Efficiency
- Cost Optimization
- Sustainability

These principles provided an initial reference for evaluating the system architecture during later implementation phases.

### Initial Architecture Baseline:

At the end of Week 1, I identified the initial logical flow of the application.

Candidate and HR users access the React/Vite frontend through a web browser. The frontend communicates with the FastAPI backend for authentication, job management, application management, document handling, and AI-related functions.

Realtime communication is handled by a separate Node.js and Socket.IO chat service. PostgreSQL stores relational application data, Redis supports caching and realtime message distribution, DynamoDB stores chat-related records, and the AI service performs CV parsing, job normalization, candidate matching, and reranking.

The initial logical architecture can be summarized as follows:

<pre>
Candidate / HR Users
        |
        v
React / Vite Frontend
        |
        +-----------------------------+
        |                             |
        v                             v
FastAPI Backend              Socket.IO Chat Service
        |                             |
        v                             v
PostgreSQL                   Redis and DynamoDB
        |
        v
Document Storage and AI Service
</pre>

This architecture was only an initial local-development baseline. It was created to understand the responsibilities and dependencies of each service before mapping them to AWS infrastructure.

The analysis also established an initial cloud migration direction:

- Host the frontend as static web content.
- Containerize the backend, chat, and AI services.
- Migrate relational data to Amazon RDS for PostgreSQL.
- Use Amazon S3 for document storage.
- Use Amazon DynamoDB for chat-related records.
- Use Redis or Amazon ElastiCache for realtime communication and caching.
- Prepare the services for automated build and deployment through CI/CD.

### Problems and Solutions:

| Problem | Root Cause | Resolution | Status |
|---|---|---|---|
| The project contained multiple services and business modules, making the overall architecture difficult to understand initially. | The repository included frontend, backend, chat, AI, database, and storage responsibilities. | I divided the system into logical service boundaries and documented the responsibility of each component. | Completed |
| I had limited practical experience with AWS IAM and cost monitoring. | Previous development work focused mainly on local application development. | I studied AWS IAM, the principle of least privilege, AWS Budgets, and basic account security practices. | Completed |
| The authentication workflow included several unfamiliar components. | Authentication required password hashing, JWT tokens, protected routes, database models, and migrations. | I reviewed the FastAPI authentication workflow step by step, from registration to protected API access. | Completed |
| No AWS deployment evidence was available during Week 1. | The first week focused on orientation, system analysis, and backend initialization. | I documented the architecture baseline and prepared the cloud deployment direction instead of claiming deployment work that had not yet been completed. | Planned |
| Screenshots and terminal logs had not yet been stored in the worklog evidence directory. | The evidence collection process had not been standardized during the first week. | I prepared an evidence checklist for AWS configuration, source code, tests, Git commits, and local application execution. | Pending |

### Testing, Build, and Deployment Results:

| Area | Result | Evidence |
|---|---|---|
| Project requirement analysis | Completed | FCAJ requirements, workshop structure, and project documentation |
| Source-code analysis | Completed | Frontend, backend, chat, AI, database, and configuration directories |
| Backend initialization | Completed | FastAPI project structure, database connection, models, and Alembic migrations |
| Authentication implementation | Completed | Registration, login, password hashing, JWT generation, and protected routes |
| Local testing | Partially completed | Backend functionality was checked locally, but no Week 1 terminal output was archived |
| Frontend build | Not yet required | Week 1 focused mainly on system analysis and backend initialization |
| AWS deployment | Planned | Cloud deployment was scheduled for later weeks |
| AWS cost monitoring | Studied and prepared | AWS Budgets documentation and budget-alert concepts |
| IAM configuration | Studied and prepared | IAM users, roles, policies, and least-privilege principles |

### Evidence to Be Added:

The following evidence should be added under:

`/images/worklog/week-1/`

Recommended evidence files:

- `project-requirements-review.png`
- `repository-structure.png`
- `backend-folder-structure.png`
- `backend-health-check.png`
- `authentication-register-test.png`
- `authentication-login-test.png`
- `alembic-migration.png`
- `aws-budget-configuration.png`
- `iam-user-or-policy.png`
- `architecture-baseline.png`
- `git-commit-history.png`

Only screenshots and logs that were actually produced during the work should be included. Missing evidence should remain marked as pending rather than being recreated or fabricated.

### Weekly Results:

By the end of Week 1, I had developed a clearer understanding of the internship requirements, the recruitment-management problem, the application source structure, and the initial AWS deployment direction.

I identified the main user groups, core functional modules, runtime services, data dependencies, and authentication flow. I also gained foundational knowledge of AWS Account management, IAM, AWS Budgets, and the AWS Well-Architected Framework.

Although no AWS workload was deployed during this week, the analysis established the technical baseline required for later containerization, networking, database migration, Kubernetes, monitoring, and CI/CD activities.

### Lessons Learned:

The main lesson from Week 1 was that cloud deployment should not begin before the application boundaries and dependencies are understood.

Although the project may appear to be a single web application, its frontend, API, realtime chat, AI processing, relational database, cache, chat storage, and document storage have different deployment and scaling requirements.

I also learned that evidence collection should take place at the same time as implementation. Screenshots, test outputs, commit hashes, build logs, and deployment results are easier to verify when they are recorded immediately instead of being collected at the end of the project.

### Next Week Plan:

The next week will focus on strengthening the application and data foundation before cloud deployment.

The planned activities include reviewing the database schema, improving data integrity, handling duplicate requests and concurrent operations, adding idempotency mechanisms, expanding backend tests, and preparing the services for Docker containerization.

The evidence collection process will also be improved by storing screenshots, test results, commit references, and build outputs under a consistent weekly directory structure.

### Week 1 Achievements:

- Understood the internship requirements and the FCAJ report structure.
- Defined the project scope and identified Candidate and HR/Company as the main user groups.
- Completed a high-level overview of the authentication, jobs, applications, documents, chat, and AI matching modules.
- Selected React/Vite, FastAPI, Node.js/Socket.IO, and an AI service for the initial system architecture.
- Learned how to create an AWS Account, configure cost alerts with AWS Budgets, and manage basic access permissions using IAM.
- Prepared the initial backend structure, database models, migrations, and authentication flow.
<!--
TODO: Add screenshots, commits, test results, AWS Budget configuration, IAM configuration, or backend testing evidence for this week.
Expected image directory:
static/images/worklog/week-1/
-->