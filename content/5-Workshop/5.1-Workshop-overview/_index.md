---
title: "Overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Objective

This workshop explains how the Internship Application Tracker is prepared, deployed, verified, monitored, troubleshot, and safely cleaned up on AWS.

The target production environment is:

| Item | Value |
|---|---|
| Project | Internship Application Tracker |
| AWS account | Project AWS account, ID redacted |
| Region | `ap-southeast-1` |
| Environment | `production` |
| EKS cluster | `internship-prod` |
| Kubernetes namespace | `internship` |
| Public frontend | `https://dhm2rz5nmsibj.cloudfront.net` |
| Application repository | `https://github.com/Temp-orgo/AWS-Internship` |

## Scope

The workshop covers the current production architecture only. The frontend is hosted on Amazon S3 and CloudFront. It is not documented as an EKS Deployment because the old Kubernetes frontend Deployment, Service, HPA, and PDB were removed.

The EKS cluster hosts these long-running workloads:

- `backend`
- `chat-service`
- `backend-outbox-dispatcher`
- `backend-processing-worker`
- `ai-service`

The managed AWS services connected to the application are RDS PostgreSQL, DynamoDB, ElastiCache Redis, S3, SQS, Lambda, SES, SageMaker, CloudFront, ALB, ECR, IAM, and CloudWatch.

## Architecture context

The browser reaches CloudFront first. CloudFront routes static frontend requests to a private S3 bucket and routes dynamic paths to the Application Load Balancer:

- Default `*` -> S3 frontend bucket
- `/api/*` -> ALB -> FastAPI backend
- `/chat/*` -> ALB -> chat service
- `/socket.io/*` -> ALB -> chat service

PostgreSQL stores transactional business data and worker queues. DynamoDB stores chat records and Lambda dedupe records. Redis provides Socket.IO pub/sub between chat pods. SQS decouples committed business events from Lambda processing. Lambda archives events to S3 and sends email through SES. The processing worker calls `ai-service`, which invokes the SageMaker endpoint `internship-qwen3-4b`.

## Target users

This workshop is written for:

- cloud engineering interns who need to explain a production AWS deployment
- backend and DevOps students deploying a multi-service application
- reviewers who need evidence-based architecture, operations, and cleanup steps
- future maintainers of the Internship Application Tracker

## Functional scope

The application supports:

- candidate and HR authentication
- job posting and job search
- candidate application submission
- application workflow management
- CV and document upload
- asynchronous CV parsing, job parsing, and matching
- realtime chat between users
- transactional outbox event publication
- email notification through Lambda and SES

## Technical scope

The technical implementation includes:

- React and Vite frontend
- FastAPI backend
- Node.js, Express, and Socket.IO chat service
- PostgreSQL with Alembic migrations
- DynamoDB chat tables
- Redis Socket.IO adapter
- EKS Deployments, Services, HPA, PDB, ServiceAccount, ConfigMap, Secret, Jobs, and Ingress
- S3 buckets for frontend assets, uploads, and event archives
- CloudFront distribution with S3 and ALB origins
- SQS Standard queue and DLQ
- Lambda event consumer with DynamoDB idempotency
- SageMaker inference endpoint behind an internal AI adapter
- GitHub Actions CI/CD using AWS OIDC

## Prerequisites

The reader should have authorized access to the AWS account, GitHub repository, EKS cluster, and production deployment settings. Exact tool versions must be verified in the target environment unless documented by the repository workflow.

## Implementation flow

1. Review the source repository and deployment scripts.
2. Confirm AWS account, region, IAM, and GitHub OIDC configuration.
3. Provision or verify AWS infrastructure.
4. Deploy RDS, Redis, DynamoDB, S3, SQS, Lambda, SageMaker, and CloudFront resources.
5. Build and push backend, chat, and AI images to ECR.
6. Deploy backend, chat, dispatcher, worker, and AI adapter to EKS.
7. Build the Vite frontend and publish it to S3.
8. Invalidate CloudFront.
9. Run health checks and end-to-end acceptance tests.
10. Monitor logs, metrics, and alarms.
11. Troubleshoot incidents using evidence.
12. Clean up resources in dependency order when the environment is no longer needed.

## Expected result

After a successful deployment:

- CloudFront serves the frontend.
- `/api/health/ready` reaches the FastAPI backend.
- `/chat/health/ready` reaches the chat service.
- Socket.IO connects through `/socket.io`.
- PostgreSQL, DynamoDB, Redis, SQS, Lambda, SES, S3, and SageMaker are integrated.
- GitHub Actions can run validation and deployment workflows through OIDC.
- The system can process business events and archive Lambda output.

## Verification

Use these high-level checks after deployment:

```bash
aws sts get-caller-identity
aws eks update-kubeconfig --name internship-prod --region ap-southeast-1
kubectl get deployments,pods,svc,hpa,pdb -n internship
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/api/health/ready
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/chat/health/ready
```

## Common errors

| Symptom | Likely cause | First diagnostic command |
|---|---|---|
| CloudFront returns frontend but API fails | ALB origin or behavior issue | `aws cloudfront get-distribution --id EQIGYNECXDYL8` |
| Backend pod not ready | PostgreSQL or secret configuration failure | `kubectl logs deployment/backend -n internship --tail=100` |
| Chat readiness returns `503` | Redis or DynamoDB unavailable | `kubectl logs deployment/chat-service -n internship --tail=100` |
| Worker stays disabled | AI deployment or SageMaker endpoint not ready | `aws sagemaker describe-endpoint --endpoint-name internship-qwen3-4b --region ap-southeast-1` |
| Lambda receives duplicates | Expected SQS at-least-once delivery | Check `InternshipLambdaEventDedupe` |

## Outcome

This overview establishes the boundary for the rest of the workshop: frontend on S3 and CloudFront, long-running services on EKS, transactional data in PostgreSQL, chat data in DynamoDB, Redis for realtime pub/sub, SQS and Lambda for event processing, and SageMaker for AI inference.
