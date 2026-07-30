---
title: "AWS Infrastructure"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

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
| Account and region | Project AWS account, `ap-southeast-1` | Account ID redacted |
| EKS | `internship-prod` | Verified from runtime evidence and workflow |
| Namespace | `internship` | Verified from manifests |
| ALB | `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` | Verified from runtime evidence |
| Frontend bucket | `internship-prod-frontend-<AWS_ACCOUNT_ID>` | Account suffix redacted |
| Upload/archive bucket | `internship-prod-uploads-<AWS_ACCOUNT_ID>` | Account suffix redacted |
| CloudFront | Distribution `EQIGYNECXDYL8` | Verified from runtime evidence |
| RDS | `internship-prod-postgres` | Verified from runtime evidence |
| Redis | Replication group `internship-prod-redis`, cluster `internship-prod-redis-001` | Verified from runtime evidence |
| DynamoDB | `ChatUsers`, `ChatGroups`, `ChatMessages`, `InternshipLambdaEventDedupe` | Verified from runtime evidence |
| SQS | `internship-prod-outbox`, `internship-prod-outbox-dlq` | Verified from runtime evidence |
| SageMaker | Endpoint `internship-qwen3-4b` | Verified from runtime evidence |
| Lambda | `internship-outbox-handler` | Verified from runtime evidence |
| IAM roles | `internship-github-deploy`, `internship-eks-runtime`, `internship-sagemaker-execution`, `internship-eks-node-role`, `internship-lambda-outbox-role`, `AmazonEKSLoadBalancerControllerRole` | Verified from deployment configuration and runtime evidence |

## Network foundation

The production EKS cluster uses a VPC created for `eksctl-internship-prod-cluster/VPC`.

| Field | Value |
|---|---|
| VPC ID | `vpc-0cfee519122ae18b4` |
| CIDR | `192.168.0.0/16` |
| DNS support | Enabled |
| DNS hostnames | Enabled |

The public subnets host internet-facing load balancer resources and NAT Gateway resources.

| Subnet ID | Availability Zone | CIDR |
|---|---|---|
| `subnet-04388ebeb4b0c8e1e` | `ap-southeast-1a` | `192.168.0.0/19` |
| `subnet-0a6b05b5878a0eebc` | `ap-southeast-1b` | `192.168.64.0/19` |
| `subnet-0311ab6100ff4ac55` | `ap-southeast-1c` | `192.168.32.0/19` |

The private subnets host EKS nodes and internal managed service access.

| Subnet ID | Availability Zone | CIDR |
|---|---|---|
| `subnet-0ee9684a5cef4c5be` | `ap-southeast-1a` | `192.168.96.0/19` |
| `subnet-049892c2d36586328` | `ap-southeast-1b` | `192.168.160.0/19` |
| `subnet-0a4831028cb96b744` | `ap-southeast-1c` | `192.168.128.0/19` |

The route table evidence shows separate public and private routing.

| Route table ID | Type | Important route |
|---|---|---|
| `rtb-0ad5499557a765f04` | Main | Local only |
| `rtb-0dd4580050faf65ce` | Public | Internet Gateway `igw-0721b83c41b0a13a7` |
| `rtb-061d062bf4bf8c9ac` | Private 1a | NAT Gateway `nat-16294630aebc49598` |
| `rtb-05fbabd9d0c5cf102` | Private 1b | NAT Gateway `nat-16294630aebc49598` |
| `rtb-0ec6c0e327db879f0` | Private 1c | NAT Gateway `nat-16294630aebc49598` |

The active NAT Gateway is `nat-16294630aebc49598`, named `internship-prod-nat`, and its runtime status is `Available`.

| Elastic IP | Allocation ID | Note |
|---|---|---|
| `54.169.85.157` | `eipalloc-0419e6eef457539f4` | Attached for `ap-southeast-1a` |
| `13.214.80.74` | `eipalloc-0c690bf3bb255dde4` | Attached for `ap-southeast-1b` |
| `47.130.245.122` | `eipalloc-013257661ce5d11d1` | Attached for `ap-southeast-1c` |
| `13.251.12.233` | `eipalloc-0a9b85e5ce92d2c02` | Unattached; should be released if unused |

The security group evidence confirms that only the public load balancer security group accepts public inbound traffic.

| Security group ID | Name | Inbound summary |
|---|---|---|
| `sg-06f3c7732550ce8fd` | EKS Control Plane | All from self and `sg-041eff3c38b13505e`; TCP `3000-8000` from load balancer security group |
| `sg-041eff3c38b13505e` | ClusterSharedNodeSG | All from self and EKS control plane security group |
| `sg-07bb82c3c3c31b61e` | `internship-prod-rds` | TCP `5432` from EKS control plane security group |
| `sg-0405e9a0dbadd61af` | `internship-prod-valkey` | TCP `6379` from EKS security group |
| `sg-0a49fc5e24439c5c3` | LoadBalancer Public | TCP `80` from `0.0.0.0/0` |
| `sg-0b5a6b3c62e503235` | LB Backend Shared | No inbound rules |

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

- private bucket `internship-prod-frontend-<AWS_ACCOUNT_ID>`
- CloudFront distribution `EQIGYNECXDYL8`
- CloudFront Origin Access Control
- default behavior to S3
- `/api/*`, `/chat/*`, `/socket.io/*` behaviors to ALB
- SPA fallback for 403 and 404 to `/index.html`

Verification:

```bash
aws s3api get-public-access-block --bucket internship-prod-frontend-<AWS_ACCOUNT_ID>
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
| Archive location | `s3://internship-prod-uploads-<AWS_ACCOUNT_ID>/outbox-archive/...` |
| Email service | Amazon SES |

Verification:

```bash
aws sqs get-queue-url --queue-name internship-prod-outbox --region ap-southeast-1
aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All --region ap-southeast-1
aws lambda get-function --function-name internship-outbox-handler --region ap-southeast-1
aws lambda list-event-source-mappings --function-name internship-outbox-handler --region ap-southeast-1
aws dynamodb describe-table --table-name InternshipLambdaEventDedupe --region ap-southeast-1
```

I recorded one naming conflict: repository scripts `scripts/aws/provision-outbox-sqs.sh` and `.ps1` default to `internship-outbox-events` with visibility timeout `60`, while the current runtime queue is `internship-prod-outbox` with visibility timeout `120`. I use runtime evidence for the deployed resource and would update the scripts only in the application repository as a separate implementation task.

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
