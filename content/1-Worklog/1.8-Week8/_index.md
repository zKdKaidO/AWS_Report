---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Week 8 - End-to-End Deployment and Operational Validation

## Objectives

Week 8 focused on the final production path for the Internship Application Tracker: building deployment images, running backend and chat workloads on Amazon EKS, exposing API and chat traffic through an Application Load Balancer, delivering the static frontend through the planned S3 and CloudFront architecture, validating AWS-managed data services, and recording which operational and cost evidence was still missing.

The report update in this page is based on the Week 08 AWS evidence files listed in the runtime and cost evidence sections. Cost values are reported from the supplied July 1-28, 2026 AWS cost summary, the billing credits screenshot, and the Cost Explorer overview screenshot. The finalized monthly bill and Pricing Calculator estimate remain unavailable.

## Tasks Completed

| Status | Task | Evidence basis |
|---|---|---|
| Completed | Added the SageMaker-oriented AI service integration and deployment path while preserving worker-facing routes. | Source references from the project implementation and `sagemaker-health.png`. |
| Completed | Fixed deploy-app image propagation and deterministic ECR image selection. | Source references from the CI/CD workflow implementation. |
| Completed | Removed frontend Kubernetes resources from the current runtime path so frontend delivery uses S3 and CloudFront instead of an EKS frontend pod. | Source references from the deployment scripts and architecture notes. |
| Completed | Verified Amazon RDS PostgreSQL and Amazon ElastiCache Valkey runtime status from AWS CLI screenshots. | `RDS-health.png`, `REDIS-health.png`. |
| Completed | Verified the SageMaker endpoint runtime status from AWS CLI screenshot evidence. | `sagemaker-health.png`. |
| Partially completed | Verified EKS workloads and ALB runtime state from AWS CLI screenshots. | EKS workloads were running, but the ALB target health screenshot showed mixed unhealthy targets. |
| Completed | Added CloudFront, S3, DynamoDB, and SQS runtime evidence. | Sanitized screenshots: `Cloudfront-evidence.png`, `S3-evidenc.png`, `DynamoDB-evidenc.png`, `SQS-evidence.png`. |
| Partially completed | Added AWS cost evidence for July 1-28, 2026. | Supplied cost summary, `total-cost-july.png`, and `Cost-evidence.png`; Pricing Calculator remains blocked. |

## Technical Implementation

The final deployment model separates static frontend hosting from long-running service workloads. Backend, chat, worker, and AI services run on EKS. Static browser assets are intended to be served through S3 and CloudFront, while API, chat, and Socket.IO paths route through the ALB.

```mermaid
flowchart LR
  Browser["User browser"] --> CF["CloudFront distribution"]
  CF --> S3["S3 frontend bucket"]
  CF --> ALB["ALB for /api, /chat, /socket.io"]
  ALB --> Backend["EKS backend"]
  ALB --> Chat["EKS chat-service"]
  Backend --> RDS["RDS PostgreSQL"]
  Chat --> Redis["ElastiCache Valkey"]
  Chat --> DDB["DynamoDB chat tables"]
  Backend --> Worker["processing worker"]
  Worker --> AI["ai-service"]
  AI --> SageMaker["SageMaker endpoint"]
  Backend --> Outbox["outbox dispatcher"]
  Outbox --> SQS["SQS queue"]
  SQS --> Lambda["Lambda notification handler"]
```

The CI/CD workflow supports deployment modes including `validate`, `deploy-app`, `deploy-public`, `deploy-frontend`, `rollout`, and `full`. Backend, chat, and AI images are tagged with the GitHub commit SHA. The frontend deployment path builds the Vite app, publishes static files to S3, and invalidates CloudFront when frontend deployment is enabled.

## AWS Runtime Evidence

The runtime evidence below uses only files that exist in the Week 08 AWS evidence directory. The reviewed files were:

- `ALB-target-health.png`
- `ALB-target.png`
- `Cloudfront-evidence.png`
- `Cost-evidence.png`
- `DynamoDB-evidenc.png`
- `eks-deployment-rollout-success.png`
- `eks-runtime-pods-2026-07-28.png`
- `lambda.png`
- `Lambda-evidence.png`
- `lambda-cloudwatch-evidence.png`
- `RDS-health.png`
- `REDIS-health.png`
- `S3-evidenc.png`
- `sagemaker-health.png`
- `SQS-evidence.png`
- `total-cost-july.png`

No `.txt`, `.log`, `.json`, `.yaml`, or `.yml` evidence file was available in this directory, so the report does not include long CLI log excerpts.

### Runtime Evidence Summary

| AWS service | Evidence status | Runtime status | Evidence |
|---|---|---|---|
| Amazon EKS | Available | Partially verified: nodes, pods, jobs, and deployments were healthy; cluster `ACTIVE` status was not directly captured | `eks-runtime-pods-2026-07-28.png`, `eks-deployment-rollout-success.png` |
| Application Load Balancer | Available | Partially operational: load balancer was active and target groups existed, but target health was mixed and included unhealthy targets | `ALB-target-health.png`, `ALB-target.png` |
| Amazon CloudFront | Available | Partially verified: monitoring screenshot shows requests, data transfer, and error-rate metrics for the distribution | `Cloudfront-evidence.png` |
| Amazon S3 | Available | Verified frontend bucket objects after account-suffix redaction | `S3-evidenc.png` |
| Amazon RDS | Available | Available | `RDS-health.png` |
| Amazon ElastiCache | Available | Available | `REDIS-health.png` |
| Amazon DynamoDB | Available | Verified: chat tables and Lambda dedupe table are Active and on-demand | `DynamoDB-evidenc.png` |
| Amazon SQS | Available | Verified: main queue and DLQ exist, show zero visible/in-flight messages, and use SSE-SQS | `SQS-evidence.png` |
| AWS Lambda | Available | Partially verified: function configuration, source view, and CloudWatch log group exist; invocation result still not verified | `lambda.png`, `Lambda-evidence.png`, `lambda-cloudwatch-evidence.png` |
| Amazon SageMaker | Available | InService | `sagemaker-health.png` |

### Amazon EKS

**Evidence status:** Available
**Runtime status:** Partially verified

The EKS screenshot evidence shows two Ready worker nodes, running application pods, completed migration/init jobs, and ready deployments in the `internship` namespace. The visible workloads include `ai-service`, `backend`, `backend-outbox-dispatcher`, `backend-processing-worker`, and `chat-service`.

The same evidence also shows deployment readiness:

- `ai-service`: 1/1 ready
- `backend`: 2/2 ready
- `backend-outbox-dispatcher`: 1/1 ready
- `backend-processing-worker`: 1/1 ready
- `chat-service`: 2/2 ready

The evidence does not include an `aws eks describe-cluster` output showing the EKS cluster status as `ACTIVE`. For that reason, this report marks EKS as partially verified instead of claiming the full cluster status was captured.

![Amazon EKS runtime pods and services](/logs/worklog/week-08/aws/eks-runtime-pods-2026-07-28.png)

![Amazon EKS deployment rollout success](/logs/worklog/week-08/aws/eks-deployment-rollout-success.png)

### Application Load Balancer

**Evidence status:** Available
**Runtime status:** Partially operational

The ALB screenshot shows an internet-facing application load balancer with state `active`. However, the target health output in the same screenshot shows mixed target status:

- One backend target on port `8000` was healthy.
- Another backend target on port `8000` was unhealthy with failed health checks.
- Chat targets on port `3000` were unhealthy with timeout or failed health-check reasons.

Because of the unhealthy targets, the ALB is not reported as fully healthy. It was deployed and active, and the target groups existed, but the target groups still required follow-up validation and remediation.

![Application Load Balancer target health](/logs/worklog/week-08/aws/ALB-target-health.png)

![Application Load Balancer target groups](/logs/worklog/week-08/aws/ALB-target.png)

### Amazon CloudFront and S3

**Evidence status:** Available
**Runtime status:** Partially verified

The CloudFront screenshot shows monitoring metrics for distribution `EQIGYNECXDYL8`, including requests, data transfer, and error-rate graphs over the selected three-day window. This confirms the distribution was receiving telemetry.

The S3 screenshot shows the frontend bucket object listing after the bucket name was redacted. The visible objects include `assets/`, `favicon.svg`, and `index.html`, which is enough to show that static frontend assets were present in S3.

These screenshots support frontend hosting evidence, but they do not replace a full CloudFront behavior/origin configuration export or a browser smoke test through the CloudFront URL.

![CloudFront monitoring evidence](/logs/worklog/week-08/aws/Cloudfront-evidence.png)

![S3 frontend bucket object evidence](/logs/worklog/week-08/aws/S3-evidenc.png)

### Amazon RDS

**Evidence status:** Available
**Runtime status:** Available

The RDS screenshot shows the PostgreSQL database instance `internship-prod-postgres` in status `available`. The captured output also shows PostgreSQL engine metadata, encrypted storage, non-public accessibility, and allocated storage.

![Amazon RDS PostgreSQL health](/logs/worklog/week-08/aws/RDS-health.png)

### Amazon ElastiCache

**Evidence status:** Available
**Runtime status:** Available

The ElastiCache screenshot shows the replication group `internship-prod-redis` with Valkey engine status `available`. The evidence also shows encryption at rest and in transit enabled.

![Amazon ElastiCache Valkey health](/logs/worklog/week-08/aws/REDIS-health.png)

### Amazon DynamoDB

**Evidence status:** Available
**Runtime status:** Verified Active

The DynamoDB screenshot shows four tables in `Active` status:

- `ChatGroups`
- `ChatMessages`
- `ChatUsers`
- `InternshipLambdaEventDedupe`

The screenshot also shows on-demand read and write capacity mode for the tables. This verifies that the AWS DynamoDB tables existed and were active at the time the evidence was captured.

![DynamoDB table status evidence](/logs/worklog/week-08/aws/DynamoDB-evidenc.png)

### Amazon SQS

**Evidence status:** Available
**Runtime status:** Verified available

The SQS screenshot shows two standard queues:

- `internship-prod-outbox`
- `internship-prod-outbox-dlq`

Both queues show zero visible messages and zero in-flight messages in the screenshot. Encryption is shown as Amazon SQS managed server-side encryption.

![SQS queue evidence](/logs/worklog/week-08/aws/SQS-evidence.png)

### AWS Lambda

**Evidence status:** Available
**Runtime status:** Partially verified

The Lambda screenshot shows a function named `internship-outbox-handler` with a Python runtime and environment configuration. The screenshot confirms that a Lambda function existed in the selected AWS region.

The screenshot was sanitized before publication by redacting account-scoped ARNs, the IAM role ARN account segment, the bucket value containing an account identifier, and the sender email address.

Additional Lambda screenshots show the function source view and a CloudWatch log group named `/aws/lambda/internship-outbox-handler`. These confirm that the function code view and log group existed. They do not prove a successful invocation, SQS trigger execution, or SES delivery result, so runtime invocation is still not marked as fully verified.

![AWS Lambda function configuration](/logs/worklog/week-08/aws/lambda.png)

![AWS Lambda source evidence](/logs/worklog/week-08/aws/Lambda-evidence.png)

![AWS Lambda CloudWatch log group evidence](/logs/worklog/week-08/aws/lambda-cloudwatch-evidence.png)

### Amazon SageMaker

**Evidence status:** Available
**Runtime status:** InService

The SageMaker screenshot shows endpoint `internship-qwen3-4b` with status `InService`. This is sufficient evidence that the SageMaker endpoint existed and was serving at the time the evidence was collected.

![Amazon SageMaker endpoint status](/logs/worklog/week-08/aws/sagemaker-health.png)

## AWS Cost Evidence

> **Status: Partially available**

The July 1-28, 2026 AWS cost summary reported a month-to-date spend of **$94.92**. The report also includes a Billing and Cost Management credits screenshot stored as `total-cost-july.png` and a Cost Explorer overview screenshot stored as `Cost-evidence.png`.

The credits screenshot shows AWS credits rather than a grouped Cost Explorer service chart. The Cost Explorer screenshot is useful as a billing-console overview, but it uses a different report configuration and is not a Pricing Calculator estimate. Therefore, the service-level cost breakdown below is still reported from the supplied July 1-28 cost summary.

![AWS Billing credits summary for July 2026](/logs/worklog/week-08/aws/total-cost-july.png)

![AWS Cost Explorer overview evidence](/logs/worklog/week-08/aws/Cost-evidence.png)

### July 1-28 cost summary

| Metric | Reported value |
|---|---:|
| Total spend, July 1-28 | $94.92 |
| Highest daily spend | $31.83 on July 28 |
| AWS credits total amount used | $27.90 |
| AWS credits total estimated amount used | $140.65 |
| AWS credits total amount remaining | $172.10 |
| AWS credits total estimated amount remaining | $59.35 |

### Daily cost trend

| Period | Daily spend |
|---|---:|
| July 1-8 | Approximately $2.20/day |
| July 9-25 | Approximately $1.60/day |
| July 26 | $5.61 |
| July 27 | $11.60 |
| July 28 | $31.83 |

The July 28 daily cost was the largest reported daily spend in the July 1-28 period and was much higher than the earlier baseline.

### Top cost drivers

| Rank | Service | Month-to-date cost | Share of total |
|---:|---|---:|---:|
| 1 | Amazon RDS | $29.69 | 31.3% |
| 2 | Amazon SageMaker | $23.45 | 24.7% |
| 3 | Amazon VPC | $14.11 | 14.9% |
| 4 | Amazon EC2 - Compute | $12.42 | 13.1% |
| 5 | EC2 - Other | $7.68 | 8.1% |
| 6 | Amazon EKS | $5.64 | 5.9% |

These six services account for approximately 98% of the July 1-28 reported spend.

### Cost anomalies

Five active anomalies were reported around July 26-28 in `ap-southeast-1`:

| Service | Anomaly impact | Reported context |
|---|---:|---|
| Amazon EBS | +$6.02, +15,050% versus expected | NAT Gateway activity in `ap-southeast-1` |
| Amazon EC2 - Compute | +$2.56, +400% versus expected | New `t3.medium` instances in `ap-southeast-1` |
| Amazon RDS | +$1.12, +138% versus expected | New `db.t4g.micro` instance activity in `ap-southeast-1` |
| Amazon VPC | +$0.47, +98% versus expected | VPC/NAT Gateway infrastructure in `ap-southeast-1` |
| AWS Secrets Manager | +$0.01, +50% versus expected | Minor secret-management cost tied to new infrastructure |

| Cost evidence | Status | Reported value |
|---|---|---:|
| July 1-28 cost summary | Available | $94.92 |
| AWS Billing credits screenshot | Available | $27.90 total amount used; $140.65 total estimated amount used |
| Cost Explorer overview screenshot | Available | Included as console evidence; not used as the final service-level breakdown |
| AWS Bills finalized monthly invoice | Pending | Not available |
| AWS Pricing Calculator | Blocked | Not available |
| Actual month-to-date cost | Reported for July 1-28 | $94.92 |
| Estimated monthly cost | Not calculated | N/A |

## Testing, Build and Deployment Results

| Area | Result | Evidence |
|---|---|---|
| EKS workload rollout | Partially verified | Deployment rollout screenshot shows all listed deployments successfully rolled out. |
| EKS runtime pods and services | Partially verified | Runtime screenshot shows Ready nodes, Running pods, Completed jobs, ready deployments, and services/ingress. |
| ALB deployment | Partially operational | ALB screenshot shows state `active`, but target health includes unhealthy targets. |
| RDS PostgreSQL | Verified available | RDS screenshot shows status `available`. |
| ElastiCache Valkey | Verified available | ElastiCache screenshot shows status `available`. |
| SageMaker endpoint | Verified InService | SageMaker screenshot shows endpoint status `InService`. |
| Lambda function listing | Partially verified | Screenshots confirm function configuration, source view, and CloudWatch log group; invocation and trigger health are not verified. |
| CloudFront and S3 frontend delivery | Partially verified | CloudFront monitoring and S3 object listing screenshots are available; full behavior/origin export and browser smoke proof remain pending. |
| DynamoDB and SQS | Verified available | DynamoDB tables are Active; SQS main queue and DLQ are listed with zero messages and SSE-SQS. |
| Cost evidence | Partially available | July 1-28 cost summary, credits screenshot, and Cost Explorer overview are available; finalized bill and Pricing Calculator remain pending/blocked. |

## Problems and Solutions

### ALB target groups were not fully healthy

**Problem:**
The ALB evidence showed the load balancer in `active` state, but target health included unhealthy backend and chat targets.

**Impact:**
API or chat traffic through the public ALB could fail intermittently or return gateway errors until every required target group becomes healthy.

**Current resolution:**
The report marks ALB as partially operational rather than fully healthy.

**Next action:**
Collect updated target health evidence after fixing health-check failures or deployment readiness issues.

### AWS cost evidence is available but not complete

**Problem:**
A July 1-28 cost summary and credits screenshot are available, but the evidence set still does not include a finalized monthly bill or AWS Pricing Calculator estimate.

**Impact:**
The report can show the July 1-28 month-to-date spend and major cost drivers, but it cannot conclude the final monthly bill or steady-state monthly cost.

**Current resolution:**
The report marks cost evidence as partially available, reports only the supplied July 1-28 figures, and keeps monthly estimate fields pending.

**Next action:**
Collect the finalized AWS Bills view and build an AWS Pricing Calculator estimate for the production architecture.

### Runtime evidence was added for several AWS services

**Problem:**
The first Week 8 evidence set did not include CloudFront, S3, DynamoDB, or SQS screenshots.

**Impact:**
Without screenshots or sanitized CLI output, those services could not be marked as runtime evidenced in the report.

**Current resolution:**
Sanitized screenshots were added for CloudFront monitoring, the S3 frontend bucket object list, DynamoDB table status, and SQS queue status.

**Next action:**
For final production evidence, add CloudFront behavior/origin configuration export and a browser smoke test through the CloudFront URL.

### Lambda screenshot required sanitization

**Problem:**
The Lambda screenshot contained account-scoped ARNs, an IAM role ARN, a bucket name containing an account identifier, and an email address.

**Impact:**
Embedding the screenshot directly would publish sensitive or semi-sensitive operational values.

**Current resolution:**
The screenshot was sanitized and embedded as Lambda deployment evidence. It confirms the function listing/configuration, but not invocation success.

**Next action:**
Collect a separate invocation, trigger, or CloudWatch event-detail screenshot if Lambda runtime execution must be marked fully verified.

## Weekly Results

Week 8 produced meaningful AWS runtime evidence for the core compute and data path:

- EKS workloads were running and deployments had rolled out.
- RDS PostgreSQL was available.
- ElastiCache Valkey was available.
- CloudFront monitoring and S3 frontend object evidence were added.
- DynamoDB tables and SQS queues were verified from AWS console screenshots.
- SageMaker endpoint evidence showed `InService`.
- Sanitized Lambda evidence confirms that the function, source view, and CloudWatch log group existed, while invocation proof remains pending.

The report also identifies important remaining gaps: ALB target health was not fully healthy, CloudFront behavior/origin export and browser smoke proof are still pending, and cost evidence still needs a finalized bill and Pricing Calculator estimate.

## Remaining Work

- [x] Add the July 1-28 AWS cost summary.
- [x] Add the AWS Billing credits screenshot.
- [x] Add a Cost Explorer overview screenshot.
- [ ] Capture the finalized AWS Billing summary.
- [ ] Build an AWS Pricing Calculator estimate. This is currently blocked.
- [x] Capture CloudFront monitoring evidence.
- [x] Capture S3 frontend bucket object evidence.
- [x] Capture Amazon DynamoDB table status evidence.
- [x] Capture Amazon SQS queue and message-count evidence.
- [ ] Capture updated ALB target health after every required target is healthy.
- [ ] Capture CloudFront behavior/origin configuration or browser smoke proof.
- [x] Add a sanitized Lambda screenshot.

## Lessons Learned

Implementation evidence and runtime evidence answer different questions. A deployment script proves that a path exists in source control, but screenshots, AWS CLI output, and health checks prove what actually ran in AWS.

Cost reporting needs the same discipline. The July 1-28 summary is useful month-to-date evidence, but the final monthly bill and steady-state monthly estimate still require separate billing and Pricing Calculator evidence.
