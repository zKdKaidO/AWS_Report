---
title: "Security and Cost"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---
# Security and Cost

## Objective

Document the security controls and cost-estimation approach for the production AWS deployment.

## Architecture context

The platform processes user accounts, CVs, job applications, chat messages, AI analysis, event notifications, and email. Security controls must protect identity, secrets, private data, deployment roles, storage, network paths, and event processing. Cost controls must pay special attention to always-on resources such as EKS, NAT Gateway, RDS, Redis, ALB, and SageMaker.

## Security controls

| Area | Implemented or required control | Status |
|---|---|---|
| GitHub OIDC | GitHub Actions assumes AWS role through OIDC with `id-token: write` | Implemented in workflow |
| Long-lived AWS keys | Do not store long-lived AWS access keys in GitHub or source code | Required control |
| IAM least privilege | Deployment, runtime, node, SageMaker, Lambda, and ALB controller roles should be scoped | Implemented concept; policy evidence required |
| IRSA | EKS ServiceAccount `internship-app` can be annotated with runtime role | Implemented in deployment script |
| Kubernetes secrets | `SECRET_KEY`, `DATABASE_URL`, `REDIS_URL`, `OUTBOX_QUEUE_URL`, and optional AI key injected as `internship-secrets` | Implemented in script |
| S3 private frontend bucket | Bucket public access blocked, CloudFront OAC used | Implemented in `ensure-cloudfront.sh`; runtime evidence required |
| CloudFront HTTPS | CloudFront redirects viewers to HTTPS | Implemented in helper config |
| ALB route control | ALB routes only API/chat/socket paths to services | Implemented in Ingress |
| RDS private access | PostgreSQL should not be publicly exposed | Evidence required |
| Redis private access | ElastiCache should not be publicly exposed | Evidence required |
| SQS SSE | Main queue and DLQ use server-side encryption | Verified from supplied context |
| DynamoDB encryption | DynamoDB tables use AWS-managed encryption by default unless configured otherwise | Evidence required for final table settings |
| RDS encryption | Must be verified with `describe-db-instances` | Evidence required |
| Encryption in transit | CloudFront HTTPS; internal service and database TLS settings require evidence | Partially evidenced |
| Secret handling | No `.env`, token, key, password, or database URL should be committed | Required control |
| DLQ | SQS DLQ isolates failed consumer messages | Verified from supplied context |
| Idempotency | PostgreSQL idempotency, outbox, chat `clientMessageId`, and Lambda DynamoDB dedupe | Implemented; Lambda smoke verified |
| Backup and PITR | RDS backup and DynamoDB PITR settings require exported evidence | Evidence required |
| Direct ALB access hardening | Restricting ALB origin to CloudFront is recommended | Optional hardening |
| AWS WAF | WAF on CloudFront is recommended for production | Optional hardening |

## Authentication and authorization

The backend uses JWT-based authentication and role-aware routes for candidate and HR workflows. The frontend stores the session under the `internship_tracker_` prefix and clears local/session storage on session expiration. Protected frontend routes redirect users according to role.

Security expectations:

- Passwords must be hashed.
- JWT secret must remain in secrets management, not in source code.
- Candidate and HR routes must enforce ownership and role checks on the backend.
- AI scores should assist review but not become the only hiring decision.

## Data protection

| Data type | Protection requirement |
|---|---|
| CVs and documents | Store in private S3, authorize access, avoid logging full content |
| Applications | Store in PostgreSQL with ownership checks |
| Chat messages | Store in DynamoDB and authorize access through chat service |
| Outbox events | Avoid future sensitive payload logging; archive only intended event content |
| AI prompts | Do not log full CV text; AI service logs prompt hash and character counts |
| Secrets | Keep in GitHub secrets and Kubernetes Secret; never publish values |

## Security verification commands

```bash
aws iam get-role --role-name internship-github-deploy
aws iam get-role --role-name internship-eks-runtime
kubectl get serviceaccount internship-app -n internship -o yaml
kubectl describe secret internship-secrets -n internship
aws s3api get-public-access-block --bucket internship-prod-frontend-587953673860
aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All --region ap-southeast-1
aws rds describe-db-instances --db-instance-identifier internship-prod-postgres --region ap-southeast-1
```

Do not print or paste secret values from Kubernetes, GitHub, or AWS into the report.

## Cost assumptions

Exact prices must come from AWS Pricing Calculator or Cost Explorer for `ap-southeast-1`. This section records the configuration and pricing inputs to gather.

| Cost driver | Known configuration | Pricing evidence required |
|---|---|---|
| EKS control plane | One cluster `internship-prod` | Monthly control-plane charge |
| EC2 worker nodes | Managed node group, type/count not in report evidence | Instance type, count, EBS volume size, runtime hours |
| NAT Gateway | Required for private subnet egress; blackhole incident confirms NAT route dependency | Hourly charge, data processing GB |
| RDS PostgreSQL | `internship-prod-postgres` | DB class, storage, IOPS if any, backup storage, Multi-AZ |
| ElastiCache Redis | `internship-prod-redis` | Node type, node count, data transfer |
| ALB | One internet-facing ALB | Runtime hours and LCU usage |
| CloudFront | Distribution `EQIGYNECXDYL8` | Requests, transfer out, invalidations |
| S3 frontend | `internship-prod-frontend-587953673860` | Storage GB, PUT/GET requests |
| S3 uploads/archive | `internship-prod-uploads-587953673860` | Storage GB, requests, lifecycle settings |
| DynamoDB | Chat and dedupe tables | Capacity mode, reads/writes, storage, PITR |
| SQS | Main queue and DLQ | Request count, payload volume |
| Lambda | `internship-outbox-handler` | Invocations, memory, duration |
| SES | Email notifications | Send count |
| ECR | Backend/chat/AI images | Storage GB and image lifecycle policy |
| SageMaker | Endpoint `internship-qwen3-4b` | Instance type, endpoint uptime, data processed |
| CloudWatch | Logs, metrics, alarms | Log ingestion, retention, custom metrics, alarms |
| Data transfer | CloudFront egress, NAT, ALB, cross-AZ if any | GB by path |

## Cost estimation table

| Service | Estimation method | Current value |
|---|---|---|
| Amazon EKS | Add one cluster for region `ap-southeast-1` in AWS Pricing Calculator | Pricing evidence required |
| EC2 worker nodes | Multiply node instance hourly price by node count and monthly runtime | Pricing evidence required |
| NAT Gateway | NAT hourly charge plus data processing | Pricing evidence required |
| Amazon RDS PostgreSQL | Instance hourly price, storage, backup, Multi-AZ if enabled | Pricing evidence required |
| Amazon ElastiCache Redis | Node hourly price times node count | Pricing evidence required |
| Application Load Balancer | ALB hourly charge plus LCU usage | Pricing evidence required |
| Amazon CloudFront | Requests plus data transfer out | Pricing evidence required |
| Amazon S3 | Storage GB plus request volume and lifecycle transitions | Pricing evidence required |
| Amazon DynamoDB | On-demand or provisioned read/write requests plus storage | Pricing evidence required |
| Amazon SQS | Standard queue requests | Pricing evidence required |
| AWS Lambda | Request count plus GB-seconds | Pricing evidence required |
| Amazon SES | Email sends and attachments if any | Pricing evidence required |
| Amazon ECR | Image storage GB and data transfer if applicable | Pricing evidence required |
| Amazon SageMaker | Endpoint instance hours and invocation/data charges | Pricing evidence required |
| Amazon CloudWatch | Log ingestion, retention, custom metrics, alarms | Pricing evidence required |
| Total | Sum of AWS Pricing Calculator export | Pricing evidence required |

## Largest cost drivers

The highest-risk cost areas are:

- SageMaker real-time endpoint uptime, especially if GPU-backed.
- EKS control plane and EC2 worker nodes running continuously.
- NAT Gateway hourly and data processing charges.
- RDS and Redis always-on managed capacity.
- ALB hourly and LCU usage.
- CloudWatch log ingestion and retention if verbose logs are kept.

SageMaker GPU uptime may dominate the total cost. If the endpoint is needed only for demos or batch processing, schedule or delete it when not used.

## Cost optimization options

| Option | Effect |
|---|---|
| Delete or schedule SageMaker endpoint when idle | Reduces likely largest AI cost driver |
| Keep processing worker disabled until AI is ready | Avoids wasted worker runtime and failed retries |
| Right-size EKS node group | Reduces EC2 and EBS cost |
| Scale non-production workloads to zero | Reduces always-on compute |
| Reduce NAT data processing | Use VPC endpoints where appropriate and minimize private subnet egress |
| S3 lifecycle rules | Move old uploads/archives to cheaper storage or expire test artifacts |
| ECR lifecycle policy | Delete old SHA images after retention window |
| CloudWatch retention | Avoid indefinite high-volume log storage |
| DynamoDB on-demand during unstable traffic | Avoids over-provisioning early |
| Serverless or asynchronous inference | Consider for workloads where real-time SageMaker endpoint uptime is unnecessary |

## Expected result

The security and cost section is complete when it identifies implemented controls, marks missing evidence clearly, avoids secret exposure, and gives a concrete cost-estimation method without inventing live AWS prices.

## Common errors

| Symptom | Cause | Resolution |
|---|---|---|
| Long-lived AWS key in CI | Static credentials used instead of OIDC | Remove key, rotate it, use OIDC role |
| Direct S3 public access | Bucket policy or public access block misconfigured | Restore public access block and CloudFront OAC policy |
| Lambda duplicate side effects | Missing event dedupe | Use DynamoDB conditional writes by `eventId` |
| Unexpected high bill | SageMaker/NAT/EKS left running | Check Cost Explorer and stop/delete idle high-cost resources |
| Cost table has fake totals | Pricing not exported | Use `Pricing evidence required` until AWS Pricing Calculator is attached |

## Outcome

The deployment follows an evidence-based security model and a practical cost model. Remaining security and cost facts require AWS CLI, Cost Explorer, or AWS Pricing Calculator exports before final numeric claims are made.
