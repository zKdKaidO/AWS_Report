---
title: "Security and Cost"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

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
| RDS private access | PostgreSQL security group allows TCP `5432` only from EKS security group | Verified from security group evidence |
| Redis private access | ElastiCache/Valkey security group allows TCP `6379` only from EKS security group | Verified from security group evidence |
| SQS SSE | Main queue and DLQ use server-side encryption | Verified from runtime evidence |
| DynamoDB encryption | `ChatGroups`, `ChatMessages`, `ChatUsers`, and `InternshipLambdaEventDedupe` use AWS owned keys | Verified |
| RDS encryption | `internship-prod-postgres` and `internship-tracker-db` use KMS encryption | Verified |
| Encryption in transit | CloudFront HTTPS; Redis transit encryption enabled and required; RDS TLS setting still requires evidence | Partially evidenced |
| Secret handling | No `.env`, token, key, password, or database URL should be committed | Required control |
| DLQ | SQS DLQ isolates failed consumer messages | Verified from runtime evidence |
| Idempotency | PostgreSQL idempotency, outbox, chat `clientMessageId`, and Lambda DynamoDB dedupe | Implemented; Lambda smoke verified |
| Backup and PITR | RDS backup retention verified; DynamoDB PITR is disabled on all listed tables | Partially verified; DynamoDB PITR recommended |
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

I do not print or paste secret values from Kubernetes, GitHub, or AWS into the documentation.

## Cost evidence source

I report cost data from the supplied AWS Budgets and runtime resource evidence. The finalized monthly bill and AWS Pricing Calculator estimate remain pending, so I do not present a full monthly price total as final.

| Budget | Limit | Actual | Forecast | Status |
|---|---:|---:|---:|---|
| `Monthly` | `$100` | `$0.00` | `$17.26` | Healthy |
| `My Monthly Cost Budget` | `$100` | `$156.02` | `$173.28` | Exceeded |

The budget named `My Monthly Cost Budget` has exceeded the configured `$100` limit. This is operational evidence that cost controls need attention before the environment is left running for a long period.

## Observed cost drivers

The supplied runtime evidence identifies these important cost drivers:

| Area | Evidence | Cost note |
|---|---|---|
| NAT Gateway | `nat-16294630aebc49598` is `Available` with three attached Elastic IPs | NAT Gateway has a fixed hourly cost plus data processing cost; review whether multi-AZ NAT is required for this internship environment |
| Unattached Elastic IP | `13.251.12.233`, allocation `eipalloc-0a9b85e5ce92d2c02` | Release if unused to avoid ongoing public IPv4 charges |
| SageMaker | Log group `/aws/sagemaker/Endpoints/internship-qwen3-4b` exists | Endpoint has run or is running; verify current endpoint status and stop/delete it when idle |
| RDS | `internship-prod-postgres` and `internship-tracker-db` both exist as `db.t4g.micro` | Review whether both databases are still required |
| EKS and worker nodes | Production EKS cluster is active | EKS control plane, EC2 nodes, EBS volumes, ALB, and logs can continue generating cost while idle |

## Cost assumptions

Exact future monthly cost still requires AWS Pricing Calculator or a representative Cost Explorer period. The deployed configuration observed in the supplied evidence is:

| Cost driver | Observed configuration | Monthly estimate input still required |
|---|---|---|
| EKS control plane | One production cluster in `ap-southeast-1` | Cluster runtime hours and current regional EKS hourly price |
| EC2 worker nodes | Production EKS worker nodes are active | Node instance type, node count, EBS size, runtime hours |
| RDS PostgreSQL | Two private, encrypted PostgreSQL `db.t4g.micro` instances, 20 GB each, Single-AZ | Review whether both instances are required; include backup storage, I/O, and runtime |
| ElastiCache / Valkey | `cache.t4g.micro`, encryption at rest and in transit, auth token enabled | Node count and data transfer |
| Application Load Balancer | One active internet-facing ALB | ALB runtime hours and LCU usage |
| CloudFront | Application distribution plus report distribution evidence | Request count, data transfer out, invalidations, behavior export |
| S3 | Frontend bucket object evidence plus upload/archive/report bucket context | Storage GB, PUT/GET requests, lifecycle policy |
| DynamoDB | Chat and Lambda dedupe tables use AWS owned key encryption; PITR disabled | Read/write request volume, storage, PITR if enabled later |
| SQS | Main outbox queue and DLQ are deployed | Request volume and payload size |
| Lambda | `internship-outbox-handler` log group exists | Invocations, average duration, errors/retries |
| SageMaker | Endpoint log group exists for `internship-qwen3-4b` | Endpoint current status, instance type, uptime, data processed |
| CloudWatch | Logs and metrics are enabled through AWS and Kubernetes services | Log ingestion, retention, custom metrics, alarms |
| Data transfer | NAT Gateway and CloudFront/ALB routing are present | CloudFront egress, ALB traffic, NAT processing, cross-AZ traffic |

## Cost estimation table

| Service | Estimation method | Current evidence status |
|---|---|---|
| Amazon EKS | Cluster hourly charge multiplied by runtime hours | Deployed; include in Pricing Calculator |
| EC2 worker nodes | Node hourly price plus EBS storage | Node instance type and EBS size still needed |
| Amazon RDS PostgreSQL | `db.t4g.micro` hourly price, 20 GB storage per instance, backup and I/O | Two private encrypted instances verified |
| Amazon ElastiCache / Valkey | `cache.t4g.micro` hourly price times node count plus data transfer | Node type verified; node count still needed |
| Application Load Balancer | ALB hourly charge plus LCU usage | Active ALB verified; target health still needs final healthy evidence |
| Amazon CloudFront | Requests plus data transfer out and invalidations | Distribution verified; request and data-transfer volume still required |
| Amazon S3 | Storage GB plus request volume and lifecycle transitions | Buckets verified; storage and request volume still required |
| Amazon DynamoDB | Read/write requests plus storage, and PITR if enabled later | Encryption verified; PITR disabled |
| Amazon SQS | Standard queue requests and payload volume | Queue and DLQ verified |
| AWS Lambda | Request count plus GB-seconds | Log group exists; invocation and trigger cost still pending |
| Amazon SES | Email send count and attachments if any | Service planned through Lambda notification path |
| Amazon ECR | Image storage GB and data transfer if applicable | Not separated in the supplied cost data |
| Amazon SageMaker | Endpoint instance hours and invocation/data charges | Endpoint log group exists; current idle/running status still needed |
| Amazon CloudWatch | Log ingestion, retention, metrics, alarms | Log groups verified; retention and ingestion trend still required |
| Amazon VPC and data transfer | CloudFront egress, ALB traffic, NAT, and cross-AZ traffic | NAT Gateway and EIPs verified; traffic breakdown still needed |
| Total | Use Pricing Calculator for forecast, then compare with representative billing evidence | Budget evidence shows one `$100` budget exceeded at `$156.02`; finalized bill and steady-state estimate pending |

## Evidence hygiene

Before I use any AWS evidence image or log in the documentation, I redact AWS account IDs, access keys, secret keys, database passwords, connection strings, Kubernetes Secret values, GitHub tokens, presigned URLs, and logs containing credentials.

My evidence scan did not find obvious access keys or GitHub tokens, but `02-eks/pod-logs-tail.txt` contains a Redis connection URL with an embedded credential from chat-service startup logs. I do not upload or paste that raw file. I use only a summary or replace the whole value with `<REDACTED_REDIS_CONNECTION_STRING>`.

## Largest cost drivers

The highest-risk future cost areas are:

- SageMaker real-time endpoint uptime, especially if the selected endpoint instance is GPU-backed.
- EKS control plane and EC2 worker nodes running continuously.
- RDS and Redis always-on managed capacity.
- ALB hourly and LCU usage.
- CloudFront and data transfer when traffic grows.
- CloudWatch log ingestion and retention if verbose logs are kept.

The budget evidence shows that cost is no longer near zero. SageMaker, RDS, VPC/NAT-related networking, EC2 worker nodes, and EKS should be treated as active cost drivers. SageMaker should be stopped or deleted when demos finish if real-time inference is not required.

The current AWS Budgets evidence is stronger than a warning: `My Monthly Cost Budget` has already exceeded its `$100` limit. The immediate cleanup candidates are the unattached Elastic IP `13.251.12.233`, idle SageMaker endpoint runtime, unnecessary duplicate RDS instances, and whether the internship environment needs three-AZ NAT Gateway coverage.

## Cost optimization options

| Option | Effect |
|---|---|
| Delete or schedule SageMaker endpoint when idle | Reduces likely largest AI cost driver |
| Keep processing worker disabled when AI testing is not active | Avoids unnecessary retries and worker runtime |
| Right-size EKS node group | Reduces EC2 and EBS cost |
| Scale non-production workloads down after demos | Reduces always-on compute |
| Review three-AZ NAT Gateway coverage | Avoids unnecessary NAT hourly and data-processing charges if full multi-AZ NAT is not needed |
| Add VPC endpoints where appropriate | Reduces NAT data processing for AWS service traffic |
| S3 lifecycle rules | Move old uploads/archives to cheaper storage or expire test artifacts |
| ECR lifecycle policy | Delete old SHA images after the retention window |
| CloudWatch retention | Avoid indefinite high-volume log storage |
| DynamoDB on-demand during unstable traffic | Avoids over-provisioning early |
| AWS Budget and billing alert | Detects unexpected charges before they become large |

## Expected result

I consider the security and cost section complete when it identifies implemented controls, marks missing evidence clearly, avoids secret exposure, and gives a concrete cost-estimation method without inventing live AWS prices.

## Common errors

| Symptom | Cause | Resolution |
|---|---|---|
| Long-lived AWS key in CI | Static credentials used instead of OIDC | Remove key, rotate it, use OIDC role |
| Direct S3 public access | Bucket policy or public access block misconfigured | Restore public access block and CloudFront OAC policy |
| Lambda duplicate side effects | Missing event dedupe | Use DynamoDB conditional writes by `eventId` |
| Unexpected high bill | SageMaker/NAT/EKS left running | Check Cost Explorer and stop/delete idle high-cost resources |
| Cost table has unsupported totals | Pricing not exported | Use AWS Pricing Calculator output before publishing a final monthly estimate |

## Outcome

The deployment follows an evidence-based security model and a practical cost model. Remaining security and cost facts require AWS CLI, Cost Explorer, or AWS Pricing Calculator exports before final numeric claims are made.
