---
title: "AWS Infrastructure"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
# AWS Infrastructure

## Objective

Document the AWS infrastructure required by the production deployment and provide safe verification commands for each major resource.

## Scope

This chapter documents implemented resources where evidence is available, identifies fields that still require exported AWS CLI evidence, and lists optional hardening separately.

## Architecture context

The infrastructure supports a hybrid edge and container architecture:

- CloudFront and S3 serve the frontend.
- The ALB routes dynamic traffic to EKS.
- EKS runs long-running application workloads.
- Managed databases, messaging, object storage, Lambda, SES, and SageMaker provide platform services.
- GitHub Actions deploys through OIDC, not through long-lived AWS access keys.

## Implemented resources

| Area | Resource | Status |
|---|---|---|
| Account and region | `587953673860`, `ap-southeast-1` | Verified from supplied context |
| EKS | `internship-prod` | Verified from supplied context and workflow |
| Namespace | `internship` | Verified from manifests |
| ALB | `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` | Verified from supplied context |
| Frontend bucket | `internship-prod-frontend-587953673860` | Verified from supplied context |
| Upload/archive bucket | `internship-prod-uploads-587953673860` | Verified from supplied context |
| CloudFront | Distribution `EQIGYNECXDYL8` | Verified from supplied context |
| RDS | `internship-prod-postgres` | Verified from supplied context |
| Redis | `internship-prod-redis`, status `available` | Verified from supplied context |
| DynamoDB | `ChatUsers`, `ChatGroups`, `ChatMessages`, `InternshipLambdaEventDedupe` | Verified from supplied context |
| SQS | `internship-prod-outbox`, `internship-prod-outbox-dlq` | Verified from supplied context |
| SageMaker | Endpoint `internship-qwen3-4b` | Verified from supplied context |
| Lambda | `internship-outbox-handler` | Verified from supplied context |
| IAM roles | `internship-github-deploy`, `internship-eks-runtime`, `internship-sagemaker-execution`, `internship-eks-node-role`, `internship-lambda-outbox-role`, `AmazonEKSLoadBalancerControllerRole` | Verified from supplied context |

## Network foundation

The production EKS cluster requires a VPC with public and private subnets:

- Public subnets host internet-facing load balancer resources and NAT Gateway resources.
- Private subnets host EKS nodes and internal managed service access.
- An Internet Gateway supports public ingress and NAT egress.
- A NAT Gateway gives private nodes outbound access for image pulls, AWS APIs, package downloads, and control-plane integration.

Evidence required:

- VPC ID, except the troubleshooting incident confirms `vpc-0cfee519122ae18b4` was used by the AWS Load Balancer Controller.
- Public subnet IDs.
- Private subnet IDs.
- Route table IDs.
- Security group IDs and rules.
- NAT Gateway ID and Elastic IP allocation ID.

Verification:

```bash
aws ec2 describe-vpcs --region ap-southeast-1
aws ec2 describe-subnets --region ap-southeast-1
aws ec2 describe-route-tables --region ap-southeast-1
aws ec2 describe-nat-gateways --region ap-southeast-1
aws ec2 describe-security-groups --region ap-southeast-1
```

## IAM and OIDC

GitHub Actions uses AWS OIDC. The workflow configures:

```yaml
permissions:
  id-token: write
  contents: read
```

Deployment jobs call `aws-actions/configure-aws-credentials@v6` with `role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}` and `allowed-account-ids: ${{ env.AWS_ACCOUNT_ID }}`.

Runtime workloads use Kubernetes ServiceAccount `internship-app`. When `IRSA_ROLE_ARN` is provided, `scripts/k8s/deploy-eks.sh` renders `k8s/eks/serviceaccount.yaml` and applies the role annotation.

Verification:

```bash
aws iam get-role --role-name internship-github-deploy
aws iam get-role --role-name internship-eks-runtime
aws iam get-role --role-name internship-lambda-outbox-role
kubectl get serviceaccount internship-app -n internship -o yaml
```

## ECR

The workflow builds SHA-tagged images and verifies backend and chat tags before deployment:

| Image | Default ECR repository |
|---|---|
| Backend | `internship-backend` |
| Chat | `internship-chat` |
| AI adapter | `internship-ai` |

Verification:

```bash
aws ecr describe-repositories --region ap-southeast-1
aws ecr describe-images --region ap-southeast-1 --repository-name internship-backend --image-ids imageTag=<GITHUB_SHA>
aws ecr describe-images --region ap-southeast-1 --repository-name internship-chat --image-ids imageTag=<GITHUB_SHA>
```

## EKS cluster and workloads

The EKS cluster hosts:

| Workload | Type | Notes |
|---|---|---|
| `backend` | Deployment and Service | FastAPI on port `8000`, 2 replicas |
| `chat-service` | Deployment and Service | Node.js/Socket.IO on port `3000`, 2 replicas |
| `backend-outbox-dispatcher` | Deployment and Service | Metrics port `9101`, 1 replica |
| `backend-processing-worker` | Deployment and Service | Metrics port `9102`, can be scaled to zero when AI is not ready |
| `ai-service` | Deployment and Service | FastAPI adapter on port `8010`, 1 replica |
| `backend-migrate` | Job | Runs `alembic upgrade head` |
| `chat-init` | Job | Initializes DynamoDB chat tables |

Verification:

```bash
aws eks describe-cluster --name internship-prod --region ap-southeast-1
aws eks list-nodegroups --cluster-name internship-prod --region ap-southeast-1
kubectl get deployments,pods,svc,hpa,pdb,jobs -n internship -o wide
```

## AWS Load Balancer Controller and ALB

The production Ingress is `internship-public` and uses AWS Load Balancer Controller annotations:

- scheme: `internet-facing`
- target type: `ip`
- listener: HTTP `80`
- health check path: `/health/ready`
- idle timeout: `3600`
- target group stickiness: enabled for `86400` seconds
- paths: `/api`, `/chat`, `/socket.io`

Verification:

```bash
kubectl get ingress internship-public -n internship -o yaml
kubectl get deployment,pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller -o wide
aws elbv2 describe-load-balancers --region ap-southeast-1
aws elbv2 describe-target-groups --region ap-southeast-1
```

## S3 and CloudFront

Frontend deployment uses:

- private bucket `internship-prod-frontend-587953673860`
- CloudFront distribution `EQIGYNECXDYL8`
- CloudFront Origin Access Control
- default behavior to S3
- `/api/*`, `/chat/*`, `/socket.io/*` behaviors to ALB
- SPA fallback for 403 and 404 to `/index.html`

Verification:

```bash
aws s3api get-public-access-block --bucket internship-prod-frontend-587953673860
aws cloudfront get-distribution --id EQIGYNECXDYL8
aws cloudfront list-invalidations --distribution-id EQIGYNECXDYL8
```

## Messaging and event-driven resources

Implemented current runtime:

| Resource | Configuration |
|---|---|
| SQS main queue | `internship-prod-outbox`, visibility timeout 120 seconds, retention 4 days, SSE enabled |
| SQS DLQ | `internship-prod-outbox-dlq`, retention 14 days, `maxReceiveCount` 5 |
| Lambda | `internship-outbox-handler`, SQS trigger |
| Dedupe table | `InternshipLambdaEventDedupe` |
| Archive location | `s3://internship-prod-uploads-587953673860/outbox-archive/...` |
| Email service | Amazon SES |

Verification:

```bash
aws sqs get-queue-url --queue-name internship-prod-outbox --region ap-southeast-1
aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All --region ap-southeast-1
aws lambda get-function --function-name internship-outbox-handler --region ap-southeast-1
aws lambda list-event-source-mappings --function-name internship-outbox-handler --region ap-southeast-1
aws dynamodb describe-table --table-name InternshipLambdaEventDedupe --region ap-southeast-1
```

Conflict to record: repository scripts `scripts/aws/provision-outbox-sqs.sh` and `.ps1` default to `internship-outbox-events` with visibility timeout `60`, while the supplied current runtime queue is `internship-prod-outbox` with visibility timeout `120`. Use runtime evidence for the deployed resource and update scripts only in the application repository if a future implementation task requests it.

## SageMaker resources

The worker should call the internal `ai-service`, and `ai-service` calls SageMaker endpoint `internship-qwen3-4b`. The deployment script requires the SageMaker endpoint to be `InService` before enabling `PROCESSING_WORKER_ENABLED=true`.

Verification:

```bash
aws sagemaker describe-endpoint --endpoint-name internship-qwen3-4b --region ap-southeast-1
kubectl get deployment,svc ai-service -n internship
```

## Proposed hardening

| Area | Hardening |
|---|---|
| ALB | Restrict direct ALB access so users enter through CloudFront where practical |
| CloudFront | Add custom domain and ACM certificate if a production domain is required |
| S3 | Confirm bucket policies allow only intended CloudFront and workload access |
| WAF | Add AWS WAF to CloudFront for common web protections |
| Logs | Set explicit CloudWatch retention for all Lambda and application log groups |
| Backups | Verify RDS backup retention and PITR requirements |
| Alarms | Add DLQ, Lambda Errors, RDS, ALB, and SageMaker cost/health alarms |
| Cost | Schedule or delete SageMaker endpoint when not in use |

## Expected result

The infrastructure is correctly prepared when the network is healthy, EKS nodes are Ready, ALB routes health checks, S3 and CloudFront serve the frontend, managed data services are reachable, SQS and Lambda process events, and SageMaker is ready before AI workers are enabled.

## Common errors

| Symptom | Root cause | Resolution |
|---|---|---|
| EKS nodes cannot join | Private route points to deleted NAT Gateway | Restore NAT and recreate/recover node group |
| ALB controller cannot infer VPC | Metadata lookup timeout | Configure `--aws-region=ap-southeast-1` and `--aws-vpc-id=vpc-0cfee519122ae18b4` |
| CloudFront API routes fail | ALB DNS missing or wrong behavior | Verify CloudFront origins and cache behaviors |
| SQS query returns queue not found | Wrong queue name | Use `internship-prod-outbox` |
| Worker fails at deploy | SageMaker endpoint not `InService` | Keep worker disabled until endpoint readiness is verified |

## Outcome

The infrastructure chapter gives the production resource map and verification commands needed before database, backend, frontend, monitoring, testing, and cleanup work can proceed.
