---
title: "Self-evaluation"
date: 2026-30-07
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Self-evaluation

## Assessment overview

This self-evaluation reflects my learning progress and contributions during the Internship Application Tracker project. The ratings are based on my actual participation in architecture analysis, AWS deployment preparation, AI integration research, troubleshooting, documentation, and validation of the deployed system. They are not official ratings from the internship supervisor.

Throughout the project, I gained practical experience in connecting application development with cloud infrastructure. I learned that deploying a complete system requires more than writing application code. It also involves networking, IAM permissions, containerization, Kubernetes workloads, managed AWS services, CI/CD workflows, monitoring, cost control, security, and clear technical documentation.

## Skill assessment

| Skill area | Self-rating | Evidence and reflection |
|---|---:|---|
| AWS architecture and service selection | 8/10 | I can explain how CloudFront, S3, ALB, EKS, RDS, Redis, DynamoDB, SQS, Lambda, SES, SageMaker, and ECR work together in the project. I also became more careful about separating the target architecture from services that were actually deployed and verified. |
| Cloud deployment and Kubernetes | 7.5/10 | I worked with the deployment structure for backend services, realtime chat, workers, and AI-related workloads on Amazon EKS. I understand the roles of Deployments, Services, health probes, autoscaling, container images, and load balancing, but I still need more hands-on experience operating EKS independently. |
| AI and machine learning integration | 8/10 | I studied how the application connects AI processing workers with a separate AI service and a SageMaker endpoint. I also worked with local language models, structured prompt design, JSON output validation, grounding, deduplication, and evaluation for job description and CV processing. I need more experience in model fine-tuning, production monitoring, and inference cost optimization. |
| Backend and event-driven design | 8/10 | I understand the use of PostgreSQL for transactional business data, Redis for fast temporary data, DynamoDB for scalable records, and SQS with Lambda for asynchronous event processing. I learned why idempotency, transactional outbox, retries, and dead-letter handling are important in distributed systems. |
| CI/CD and automation | 7.5/10 | I learned how GitHub Actions, AWS OIDC, Amazon ECR, automated image builds, deployment workflows, and rollout verification support repeatable releases. I can follow and document the workflow, but I need more practice designing a complete pipeline from scratch. |
| Security and IAM awareness | 7/10 | I understand the importance of avoiding long-lived access keys, using OIDC and IAM roles, separating secrets, restricting S3 access, encrypting queues, and applying least-privilege permissions. My next improvement is learning how to review and design IAM policies more systematically. |
| Troubleshooting and system validation | 8.5/10 | One of my strongest improvements was learning to investigate problems using logs, command output, deployment status, and runtime evidence. I worked through issues involving networking, EKS nodes, ALB configuration, GitHub Actions conditions, SQS integration, CloudFront routing, and AWS resource configuration. |
| Technical documentation and communication | 8.5/10 | I organized project information into an architecture proposal, deployment workshop, weekly worklog, testing evidence, troubleshooting notes, security analysis, cost analysis, and final report. I improved at explaining technical processes in a structured and reproducible way. |

## Key strengths

- I can connect application code, cloud services, deployment configuration, and runtime evidence into one consistent system explanation.
- I approach technical problems by checking logs and actual system behavior instead of relying only on assumptions.
- I can research AI integration approaches and translate them into practical backend workflows.
- I am careful about distinguishing completed implementation, successful deployment, planned architecture, and future improvements.
- I can document complex deployment procedures in a way that another person can follow and reproduce.
- I understand the main trade-offs among system reliability, security, scalability, performance, and AWS cost.

## Main challenges

- Amazon EKS has many interconnected components, so errors may come from Kubernetes, IAM, networking, load balancers, container images, or AWS controllers rather than from application code.
- AI model deployment requires balancing accuracy, response time, GPU resources, endpoint availability, and operating cost.
- Some AWS services are expensive when kept running continuously, especially EKS infrastructure, NAT Gateway, databases, and SageMaker endpoints.
- IAM troubleshooting can be difficult because a deployment may require permissions across several services and execution roles.
- Collecting reliable evidence requires checking both configuration and actual runtime results rather than depending only on architecture diagrams or source code.
- Working in a shared repository requires careful Git operations to avoid overwriting another member's work or committing private local files.

## Lessons learned

- A good architecture diagram should reflect the implementation and clearly label services that are only proposed.
- Runtime logs, screenshots, CLI output, and successful tests provide stronger evidence than configuration files alone.
- Distributed systems must expect duplicate messages, retries, partial failures, and temporary service unavailability.
- AI integration should use a stable application-facing interface so that the underlying model or deployment method can be changed later.
- Kubernetes health probes and autoscaling only work correctly when resource requests, dependencies, and application behavior are configured properly.
- Security should be considered during architecture design rather than added only after deployment.
- Cost optimization requires reviewing resource uptime, data transfer, storage, logs, and managed-service pricing together.
- Technical documentation is part of the engineering process because it supports deployment, troubleshooting, knowledge transfer, and evaluation.

## Areas for improvement

- Gain more independent hands-on experience with EKS cluster creation, VPC design, ingress configuration, autoscaling, and observability.
- Improve IAM policy design using least privilege, permission boundaries, resource-level permissions, and policy analysis tools.
- Study model evaluation methods for job and CV matching, including dataset construction, retrieval metrics, reranking quality, and human evaluation.
- Learn more about model fine-tuning, quantization, asynchronous inference, endpoint autoscaling, and lower-cost AI serving options.
- Build automated integration and end-to-end tests for login, job posting, job application, CV processing, realtime chat, and notification flows.
- Improve monitoring by adding meaningful CloudWatch dashboards, alarms, tracing, failure-rate metrics, and cost alerts.
- Practice disaster recovery planning, database backup validation, point-in-time recovery, and incident response procedures.

## Future development plan

1. Build a smaller AWS environment independently to practice VPC, EKS, ALB, RDS, IAM, and CI/CD configuration from the beginning.
2. Continue developing the AI processing pipeline for job descriptions and CVs, with structured outputs, grounding checks, reranking, and measurable evaluation.
3. Add automated smoke tests and browser-based end-to-end tests for the main Candidate and HR workflows.
4. Improve observability through centralized logs, metrics, dashboards, alarms, and traceable request identifiers.
5. Compare SageMaker endpoints with alternative inference approaches based on accuracy, latency, scalability, and monthly cost.
6. Prepare a complete deployment evidence package containing screenshots, CLI output, build logs, runtime logs, test results, and AWS cost information.

## Career orientation

This internship project strengthened my interest in cloud engineering, backend development, DevOps, and AI-enabled systems. I am particularly interested in roles that combine software development with cloud infrastructure, automation, system reliability, and practical AI integration.

In the future, I want to become capable of designing and deploying a complete system independently, from application architecture and AI processing to infrastructure, security, monitoring, testing, and cost optimization.