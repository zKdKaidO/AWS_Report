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
aws s3api get-public-access-block --bucket <FRONTEND_BUCKET_NAME>
aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All --region ap-southeast-1
aws rds describe-db-instances --db-instance-identifier internship-prod-postgres --region ap-southeast-1
```

Do not print or paste secret values from Kubernetes, GitHub, or AWS into the report.

## Cost evidence source

Cost data was exported from Cost Explorer in the evidence folder `aws-evidence-2026-07-29/01-cost`. The command used the Cost Explorer API endpoint in `us-east-1`, while the deployed workload evidence is for `ap-southeast-1`.

| Billing period | Cost Explorer metric | Observed total | Interpretation |
|---|---|---:|---|
| 2026-06-01 to 2026-07-01 | UnblendedCost | `-0.0000001996 USD` | Effectively zero; includes tiny credits or rounding adjustments |
| 2026-07-01 to 2026-07-30 | UnblendedCost | `0.0000000871 USD` | Effectively zero for the captured month-to-date window |

These values are billing evidence for the exported period only. They should not be treated as a steady-state monthly production estimate because the AWS resources were recently created, several services show zero or tiny usage, and credits or negative adjustments can cancel gross charges.

## Observed current-month service cost

The current month-to-date Cost Explorer export shows the following service-level values.

| Service | Current month-to-date UnblendedCost |
|---|---:|
| EC2 - Other | `0.0293361372 USD` |
| AWS Data Transfer | `-0.0293422518 USD` |
| Amazon Elastic Compute Cloud - Compute | `0.0000028030 USD` |
| AWS CloudShell | `0.0000015348 USD` |
| Elastic Load Balancing | `0.0000012880 USD` |
| Amazon Simple Storage Service | `0.0000004694 USD` |
| Amazon Relational Database Service | `0.0000000584 USD` |
| AWS Secrets Manager | `0.0000000376 USD` |
| Amazon Elastic Container Registry Public | `0.0000000071 USD` |
| Amazon CloudWatch | `0.0000000034 USD` |
| Amazon DynamoDB | `0.0000000000 USD` |
| Amazon ElastiCache | `0.0000000000 USD` |
| Amazon Elastic Container Service for Kubernetes | `0.0000000000 USD` |
| AWS Lambda | `0.0000000000 USD` |
| Amazon SageMaker | `0.0000000000 USD` |
| Amazon Simple Queue Service | `0.0000000000 USD` |
| Amazon Simple Email Service | `0.0000000000 USD` |

The largest observed gross line items are `EC2 - Other` and the negative `AWS Data Transfer` adjustment, which nearly cancel each other. The zero entries are useful evidence that the Cost Explorer export did not yet capture meaningful billable usage for those services; they are not proof that those services are free in normal operation.

## Cost assumptions

Exact future monthly cost still requires AWS Pricing Calculator or a representative Cost Explorer period. The deployed configuration observed in the evidence folder is:

| Cost driver | Observed configuration | Monthly estimate input still required |
|---|---|---|
| EKS control plane | One production cluster in `ap-southeast-1` | Cluster runtime hours and current regional EKS hourly price |
| EC2 worker nodes | Two Ready Kubernetes worker nodes | Node instance type, EBS size, runtime hours |
| RDS PostgreSQL | Two private, encrypted PostgreSQL `db.t4g.micro` instances, 20 GiB each, Single-AZ | Backup storage, additional I/O, representative runtime |
| ElastiCache / Valkey | One available replication group with encryption at rest and in transit | Node type, node count, data transfer |
| Application Load Balancer | One active internet-facing ALB | ALB runtime hours and LCU usage |
| CloudFront | Two deployed distributions, including the application distribution and the report distribution | Request count, data transfer out, invalidations |
| S3 | Frontend, upload/archive, report, and support buckets in AWS evidence | Storage GB, PUT/GET requests, lifecycle policy |
| DynamoDB | Chat and Lambda dedupe tables use on-demand billing; table sizes are near zero in the snapshot | Read/write request volume, storage, PITR if enabled |
| SQS | Main queue and DLQ had zero visible messages in the snapshot | Request volume and payload size |
| Lambda | Outbox handler is Active, 256 MB memory, 20 second timeout | Invocations, average duration, errors/retries |
| SageMaker | Real-time endpoint is `InService` with one production variant | Instance type, endpoint uptime, data processed |
| CloudWatch | Logs and metrics are enabled through AWS and Kubernetes services | Log ingestion, retention, custom metrics, alarms |
| Data transfer | Cost Explorer shows data transfer adjustment in the captured period | CloudFront egress, ALB traffic, NAT processing, cross-AZ traffic |

## Cost estimation table

| Service | Estimation method | Current evidence status |
|---|---|---|
| Amazon EKS | Cluster hourly charge multiplied by runtime hours | Deployed; current Cost Explorer line is zero in captured window |
| EC2 worker nodes | Node hourly price times two nodes plus EBS storage | Nodes verified; instance type and volume size still needed |
| Amazon RDS PostgreSQL | `db.t4g.micro` hourly price, 20 GiB storage per instance, backup and I/O | Two private encrypted instances verified |
| Amazon ElastiCache / Valkey | Node hourly price times node count plus data transfer | Replication group verified; node type/count still needed |
| Application Load Balancer | ALB hourly charge plus LCU usage | Active ALB verified; tiny current Cost Explorer line |
| Amazon CloudFront | Requests plus data transfer out and invalidations | Distributions verified; current Cost Explorer line is zero |
| Amazon S3 | Storage GB plus request volume and lifecycle transitions | Buckets verified; tiny current Cost Explorer line |
| Amazon DynamoDB | On-demand read/write requests plus storage | Tables verified; current Cost Explorer line is zero |
| Amazon SQS | Standard queue requests and payload volume | Queue and DLQ verified; current Cost Explorer line is zero |
| AWS Lambda | Request count plus GB-seconds | Function verified; current Cost Explorer line is zero |
| Amazon SES | Email send count and attachments if any | Service planned through Lambda notification path |
| Amazon ECR | Image storage GB and data transfer if applicable | Tiny current Cost Explorer line |
| Amazon SageMaker | Endpoint instance hours and invocation/data charges | Endpoint verified `InService`; current Cost Explorer line is zero |
| Amazon CloudWatch | Log ingestion, retention, metrics, alarms | Tiny current Cost Explorer line |
| Data transfer | CloudFront egress, ALB traffic, NAT, and cross-AZ traffic | Adjustment observed; traffic breakdown still needed |
| Total | Use Pricing Calculator for forecast, then compare with representative Cost Explorer | Current MTD observed total is effectively zero |

## Evidence hygiene

Before using any AWS evidence image or log in the report, redact AWS account IDs, access keys, secret keys, database passwords, connection strings, Kubernetes Secret values, GitHub tokens, presigned URLs, and logs containing credentials.

The evidence scan did not find obvious access keys or GitHub tokens, but `02-eks/pod-logs-tail.txt` contains a Redis connection URL with an embedded credential from chat-service startup logs. Do not upload or paste that raw file. Use only a summary or replace the whole value with `<REDACTED_REDIS_CONNECTION_STRING>`.

## Largest cost drivers

The highest-risk future cost areas are:

- SageMaker real-time endpoint uptime, especially if the selected endpoint instance is GPU-backed.
- EKS control plane and EC2 worker nodes running continuously.
- RDS and Redis always-on managed capacity.
- ALB hourly and LCU usage.
- CloudFront and data transfer when traffic grows.
- CloudWatch log ingestion and retention if verbose logs are kept.

The current Cost Explorer export is near zero, but a continuously running production environment will still accrue normal service charges. SageMaker should be stopped or deleted when demos finish if real-time inference is not required.

## Cost optimization options

| Option | Effect |
|---|---|
| Delete or schedule SageMaker endpoint when idle | Reduces likely largest AI cost driver |
| Keep processing worker disabled when AI testing is not active | Avoids unnecessary retries and worker runtime |
| Right-size EKS node group | Reduces EC2 and EBS cost |
| Scale non-production workloads down after demos | Reduces always-on compute |
| Confirm whether NAT Gateway is required | Avoids NAT hourly and data-processing charges if private egress is not needed |
| Add VPC endpoints where appropriate | Reduces NAT data processing for AWS service traffic |
| S3 lifecycle rules | Move old uploads/archives to cheaper storage or expire test artifacts |
| ECR lifecycle policy | Delete old SHA images after the retention window |
| CloudWatch retention | Avoid indefinite high-volume log storage |
| DynamoDB on-demand during unstable traffic | Avoids over-provisioning early |
| AWS Budget and billing alert | Detects unexpected charges before they become large |

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
