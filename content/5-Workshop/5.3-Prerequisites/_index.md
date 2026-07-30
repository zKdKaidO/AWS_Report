---
title: "Prerequisites"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Objective

Prepare the accounts, permissions, tools, repository access, quotas, and environment variables required to deploy and verify the Internship Application Tracker.

## Architecture context

This project uses GitHub Actions and AWS OIDC for production deployment. Local tools are still needed for inspection, emergency verification, and manual recovery. Do not use long-lived AWS access keys in the CI/CD pipeline.

## Required access

| Access area | Requirement |
|---|---|
| AWS account | Authorized access to the project AWS account; account ID redacted |
| AWS region | `ap-southeast-1` |
| GitHub repository | Access to `https://github.com/Temp-orgo/AWS-Internship` |
| GitHub Actions environment | Production environment secrets and variables |
| EKS | Permission to update kubeconfig and inspect namespace `internship` |
| CloudFront and S3 | Permission to inspect distribution, bucket policy, objects, and invalidations |
| RDS, Redis, DynamoDB, SQS, Lambda, SES, SageMaker | Read access for validation; deployment role needs scoped write actions |

## Required tools

| Tool | Required version or configuration | Evidence |
|---|---|---|
| Git | Version must be verified in the target environment. | Local shell |
| AWS CLI | Version must be verified in the target environment. | Deployment scripts call `aws` |
| kubectl | Version must be verified in the target environment. | GitHub Actions uses `azure/setup-kubectl@v5` |
| Helm | Version must be verified in the target environment. | Local Kubernetes runbook uses Helm |
| Docker | Version must be verified in the target environment. | CI builds Docker images |
| Node.js | `22` in `.github/workflows/cicd.yml`; local version must be verified. | Workflow env `NODE_VERSION` |
| npm | Comes with the local or CI Node.js runtime. | `npm ci`, `npm run build`, `npm run test` |
| Python | `3.12` in `.github/workflows/cicd.yml`; local version must be verified. | Workflow env `PYTHON_VERSION` |
| pip | Required for backend dependencies. | `backend/requirements.txt` |
| jq | Required by CloudFront helper script. | `scripts/aws/ensure-cloudfront.sh` |
| Hugo Extended | `0.134.3` in the report GitHub Actions workflow; local version must be verified. | Report workflow |
| Terraform | Not found as an implementation dependency in the inspected repository. | Infrastructure uses YAML and scripts |

## AWS resources that must already exist or be created

| Resource | Required value or status |
|---|---|
| EKS cluster | `internship-prod` |
| Kubernetes namespace | `internship` |
| Frontend bucket | `internship-prod-frontend-<AWS_ACCOUNT_ID>` |
| Upload/archive bucket | `internship-prod-uploads-<AWS_ACCOUNT_ID>` |
| CloudFront distribution | `EQIGYNECXDYL8` |
| ALB DNS | `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` |
| RDS PostgreSQL | `internship-prod-postgres` |
| ElastiCache Redis | `internship-prod-redis` |
| DynamoDB chat tables | `ChatUsers`, `ChatGroups`, `ChatMessages` |
| DynamoDB Lambda dedupe table | `InternshipLambdaEventDedupe` |
| SQS queue | `internship-prod-outbox` |
| SQS DLQ | `internship-prod-outbox-dlq` |
| SageMaker endpoint | `internship-qwen3-4b` |
| Lambda function | `internship-outbox-handler` |

## AWS quotas to check

| Service | Quota to check | Why it matters |
|---|---|---|
| EKS | Cluster and managed node group quotas | Required for workload scheduling |
| EC2 | vCPU, Elastic IP, NAT Gateway, ENI quotas | EKS nodes, ALB targets, and NAT depend on these |
| ALB | Load balancer and target group quotas | Ingress controller creates ALB resources |
| RDS | DB instances and storage | PostgreSQL deployment |
| ElastiCache | Node quota | Redis replication group |
| DynamoDB | Table and account throughput quotas | Chat and Lambda dedupe tables |
| SQS | Queue and message throughput | Outbox event delivery |
| Lambda | Concurrency | SQS event consumer |
| SES | Sending sandbox or production sending limit | Email notification smoke tests |
| SageMaker | Endpoint instance quota, especially GPU quota if used | AI endpoint uptime and capacity |
| CloudFront | Distribution and invalidation quotas | Frontend delivery and deployments |

## GitHub Actions variables and secrets

The workflow reads deployment values from GitHub environment variables and secrets. Values below are names only and must not be published with secret values.

### Required or commonly used variables

| Variable | Purpose |
|---|---|
| `AWS_REGION` | Target region, default `ap-southeast-1` |
| `AWS_ACCOUNT_ID` | Target account ID, set from the project AWS account |
| `EKS_CLUSTER_NAME` | EKS cluster, default `internship-prod` |
| `K8S_NAMESPACE` | Namespace, default `internship` |
| `EKS_DEPLOY_ENABLED` | Must be `true` for app deployment |
| `ENABLE_ALB_INGRESS` | Controls public ALB Ingress deployment |
| `FRONTEND_DEPLOY_ENABLED` | Must be `true` for frontend deployment |
| `CREATE_CLOUDFRONT` | Allows CloudFront creation when distribution ID is absent |
| `FRONTEND_BUCKET` | Frontend bucket |
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront distribution ID |
| `ALB_DNS` | Existing ALB origin when needed |
| `ECR_REPOSITORY_BACKEND` | Backend ECR repository, default `internship-backend` |
| `ECR_REPOSITORY_CHAT` | Chat ECR repository, default `internship-chat` |
| `ECR_REPOSITORY_AI` | AI ECR repository, default `internship-ai` |
| `AI_DEPLOY_ENABLED` | Controls AI adapter deployment |
| `PROCESSING_WORKER_ENABLED` | Controls processing worker scale-up |
| `SAGEMAKER_ENDPOINT_NAME` | SageMaker endpoint, default `internship-qwen3-4b` |
| `IRSA_ROLE_ARN` | Runtime IAM role annotation |

### Required secrets

| Secret | Purpose |
|---|---|
| `AWS_ROLE_TO_ASSUME` | GitHub OIDC deployment role |
| `SECRET_KEY` | Backend JWT/application secret |
| `DATABASE_URL` | PostgreSQL connection URL |
| `REDIS_URL` | Redis connection URL |
| `OUTBOX_QUEUE_URL` | SQS main queue URL |
| `AI_SERVICE_API_KEY` | Optional shared key for worker-to-AI calls |

## Verification commands

Run these commands only with authorized credentials:

```bash
aws sts get-caller-identity
aws configure get region
aws eks update-kubeconfig --name internship-prod --region ap-southeast-1
kubectl get namespace internship
kubectl get nodes -o wide
kubectl get deployments -n internship
```

Check tool versions:

```bash
git --version
aws --version
kubectl version --client
helm version
docker version
node --version
npm --version
python --version
jq --version
hugo version
```

## Expected result

- The AWS account ID is the project account ID and should be redacted from published screenshots or logs.
- The active region is `ap-southeast-1`.
- `kubectl` can inspect the `internship` namespace.
- GitHub Actions can assume the deploy role through OIDC.
- Required secrets are stored in GitHub environment secrets, not in source control.

## Common errors

| Symptom | Cause | Resolution |
|---|---|---|
| `AccessDenied` during deployment | Missing IAM action on the OIDC role | Add least-privilege permission for the specific API call |
| `kubectl` connects to the wrong cluster | Active kube context is not production | Run `kubectl config current-context` and update kubeconfig |
| Frontend deployment skipped | `FRONTEND_DEPLOY_ENABLED` is false or only configured as an environment variable unavailable to a job-level `if` | Set the required flag as a GitHub repository or environment variable as used by the workflow |
| Worker deployment fails | `PROCESSING_WORKER_ENABLED=true` without AI readiness | Enable worker only after AI service image and SageMaker endpoint are ready |
| `jq is required` | `jq` missing from local environment | Install `jq` before running CloudFront helper locally |

## Outcome

The environment is ready for source-code preparation and deployment only after account access, tools, OIDC variables, secrets, quotas, and Kubernetes access have been verified.
