---
title: "Tự đánh giá"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Self-evaluation

## Assessment summary

This is my draft self-assessment based on the Internship Application Tracker project. It is not an official supervisor rating.

During the project, I worked on a cloud-native internship application platform that combines a React/Vite frontend, FastAPI backend, Node.js Socket.IO chat service, PostgreSQL, DynamoDB, Redis, SQS, Lambda, S3, CloudFront, EKS, and SageMaker. The most important learning outcome for me was understanding how application code, Kubernetes, managed AWS services, IAM, networking, CI/CD, and runtime evidence connect in a real deployment.

## Skill rating table

| Skill area | Draft rating | Evidence and reflection |
|---|---:|---|
| AWS architecture | 8/10 | I helped shape the final architecture where CloudFront routes static frontend traffic to S3 and dynamic `/api`, `/chat`, and `/socket.io` traffic to the ALB. I learned to distinguish proposal diagrams from deployed evidence. |
| Kubernetes | 8/10 | I worked with EKS workloads for backend, chat, outbox dispatcher, processing worker, and ai-service, including Deployments, Services, readiness/liveness probes, HPA, and PDB. |
| Networking | 7/10 | I investigated issues such as a NAT Gateway blackhole that prevented EKS nodes from joining and the AWS Load Balancer Controller VPC ID failure. I still need more practice designing VPCs from scratch. |
| IAM and security | 7/10 | I used GitHub OIDC, IRSA, runtime roles, secret separation, S3 private access, SQS SSE, and idempotency. I need to improve formal IAM policy review and permission boundary design. |
| CI/CD | 8/10 | I worked with GitHub Actions workflow modes such as `validate`, `deploy-app`, `deploy-public`, `deploy-frontend`, `rollout`, and `full`, including ECR SHA image verification and frontend S3 deployment. |
| Database and messaging | 8/10 | I learned how PostgreSQL stores business data, processing jobs, idempotency records, and transactional outbox events, while SQS carries committed events to Lambda and DynamoDB supports chat and dedupe. |
| Serverless | 7/10 | I integrated the SQS-to-Lambda notification flow conceptually and documented the successful Lambda smoke test, DynamoDB dedupe, S3 archive, SES result, and partial batch failure behavior. |
| AI integration | 7/10 | I worked with the processing worker and ai-service adapter pattern, where the worker keeps stable routes and the adapter calls SageMaker endpoint `internship-qwen3-4b`. I need more experience operating model endpoints cost-effectively. |
| Troubleshooting | 8/10 | I practiced evidence-led debugging for NAT, ALB controller, GitHub Actions job conditions, SQS queue naming, CloudFront routing, and DynamoDB reserved keyword errors. |
| Documentation | 8/10 | I converted implementation details and runtime context into workshop, proposal, cost, security, testing, troubleshooting, and cleanup documentation. |

## Strengths

- I can connect source code, deployment manifests, workflow files, and runtime evidence into one coherent architecture explanation.
- I became more careful about distinguishing implemented code from verified production behavior.
- I improved at troubleshooting AWS integration failures by starting from logs and command output instead of assumptions.
- I learned how to document deployment procedures with warnings, expected results, and common errors.
- I understand why frontend static hosting, EKS services, database transactions, SQS delivery, Lambda idempotency, and SageMaker inference belong in different parts of the architecture.

## Challenges

- AWS service interactions can fail for reasons outside application code, especially IAM, route tables, controller configuration, and GitHub Actions variable scope.
- Debugging EKS required understanding both Kubernetes objects and AWS-created resources such as ALB target groups and ENIs.
- Cost estimation was difficult because exact AWS prices depend on live instance types, endpoint uptime, data transfer, and log retention.
- AI integration required preserving the worker-facing contract while moving implementation details into the SageMaker adapter.
- Documentation required discipline because it is easy to overstate something that exists in code but has not been verified in production.

## Lessons learned

- Runtime evidence should be treated as stronger than old manifests when describing current production architecture.
- A Kubernetes Service load-balances traffic but does not serialize database writes; correctness must come from constraints, idempotency, conditional writes, and transactions.
- SQS Standard provides at-least-once delivery, so consumers must deduplicate events.
- Lambda is useful for short event-driven processing, but it only reduces cost when it removes or scales down always-on capacity.
- Static frontend delivery through S3 and CloudFront is a better fit than running a frontend pod in EKS for this project.
- Worker workloads should be enabled only after their dependencies, especially SageMaker, are ready.

## Areas for improvement

- Practice writing IAM policies with least privilege from the beginning instead of tightening them after deployment failures.
- Learn more about VPC endpoint strategy to reduce NAT Gateway cost and dependency.
- Add stronger production evidence collection for alarms, backups, encryption settings, and PITR.
- Build a repeatable frontend/browser E2E test harness for login, job apply, chat, and AI processing flows.
- Improve SageMaker cost control by testing scheduled shutdown, asynchronous inference, or lower-cost inference options.

## Future plan

1. Strengthen AWS networking knowledge, especially VPC routing, NAT, endpoints, security groups, and private service access.
2. Practice production IAM design with OIDC, IRSA, Lambda roles, and SageMaker execution roles.
3. Build a complete evidence package for deployed systems: CLI exports, logs, screenshots, Cost Explorer, and Pricing Calculator results.
4. Add automated E2E tests and smoke tests for the main candidate and HR flows.
5. Continue improving event-driven design with SQS, Lambda, DLQ handling, idempotency, and observability.

## Career orientation

This project confirmed that I want to keep developing skills in cloud engineering, backend systems, DevOps, and AI-enabled applications. I am especially interested in work that combines application development with reliable infrastructure, security, automation, and practical troubleshooting.
