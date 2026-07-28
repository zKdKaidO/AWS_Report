# Internship Application Tracker — Project Context

## 1. Project information

- Project name: Internship Application Tracker
- Repository: https://github.com/Temp-orgo/AWS-Internship
- AWS account ID: 587953673860
- AWS region: ap-southeast-1
- Kubernetes namespace: internship
- EKS cluster: internship-prod
- Deployment environment: production
- Main deployment method: GitHub Actions + GitHub OIDC
- Frontend framework: React/Vite
- Backend framework: FastAPI
- Chat service: Node.js + Socket.IO
- Primary database: Amazon RDS PostgreSQL
- Chat storage: Amazon DynamoDB
- Realtime adapter: Amazon ElastiCache Redis
- AI inference: Amazon SageMaker endpoint
- Container orchestration: Amazon EKS

## 2. Main application capabilities

The system supports:

1. User authentication and account management.
2. HR job posting management.
3. Candidate job search and job application.
4. Candidate application status management.
5. CV and attachment uploads.
6. Asynchronous CV parsing and AI processing.
7. Realtime chat through Socket.IO.
8. Transactional outbox event publishing.
9. Notification processing with AWS Lambda and Amazon SES.
10. Static frontend delivery through S3 and CloudFront.

## 3. Current AWS architecture

### Edge and frontend

- CloudFront distribution ID: EQIGYNECXDYL8
- CloudFront domain: dhm2rz5nmsibj.cloudfront.net
- Frontend S3 bucket: internship-prod-frontend-587953673860
- Upload and audit bucket: internship-prod-uploads-587953673860

CloudFront behaviors:

- Default `*` routes to the frontend S3 origin.
- `/api/*` routes to the Application Load Balancer.
- `/chat/*` routes to the Application Load Balancer.
- `/socket.io/*` routes to the Application Load Balancer.

### Load balancing

- ALB DNS:
  k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com
- ALB type: internet-facing
- Target type: IP
- Backend target port: 8000
- Chat target port: 3000
- Health check path: /health/ready
- Idle timeout: 3600 seconds
- Target group stickiness: enabled

### Amazon EKS workloads

The EKS cluster hosts:

- backend
- chat-service
- backend-outbox-dispatcher
- backend-processing-worker
- ai-service

Confirmed scaling configuration:

- backend: 2 replicas
- chat-service: 2 replicas
- backend-outbox-dispatcher: 1 replica
- backend-processing-worker: enabled after SageMaker deployment
- frontend Kubernetes resources were removed because frontend is hosted by S3

Resource requests and limits:

Backend:
- CPU request: 100m
- Memory request: 192Mi
- CPU limit: 750m
- Memory limit: 768Mi

Chat service:
- CPU request: 100m
- Memory request: 160Mi
- CPU limit: 750m
- Memory limit: 640Mi

HPA:

- backend: min 2, max 5, CPU target 70%
- chat-service: min 2, max 5, CPU target 70%

PDB:

- backend minimum available: 1
- chat-service minimum available: 1

### Databases and storage

Amazon RDS:

- Identifier: internship-prod-postgres
- Engine: PostgreSQL
- Purpose:
  - users
  - jobs
  - applications
  - processing jobs
  - transactional outbox
  - idempotency records
  - application business data

Amazon ElastiCache:

- Replication group: internship-prod-redis
- Status: available
- Purpose: Socket.IO pub/sub between multiple chat pods

Amazon DynamoDB tables:

- ChatUsers
- ChatGroups
- ChatMessages
- InternshipLambdaEventDedupe

The three chat tables were verified as ACTIVE.

Amazon S3:

- internship-prod-frontend-587953673860
  - frontend static files
- internship-prod-uploads-587953673860
  - CV uploads
  - attachments
  - Lambda event archive

### Messaging

Main SQS queue:

- internship-prod-outbox
- Visibility timeout: 120 seconds
- Message retention: 4 days
- Server-side encryption: enabled

Dead-letter queue:

- internship-prod-outbox-dlq
- Message retention: 14 days
- maxReceiveCount: 5

The outbox dispatcher publishes messages from PostgreSQL outbox records to SQS.

### AI layer

- SageMaker endpoint name: internship-qwen3-4b
- Model family: Qwen3 4B
- AI service acts as an adapter between EKS worker and SageMaker.
- Processing worker claims jobs from PostgreSQL.
- Worker sends inference requests through ai-service.
- Results are persisted back into PostgreSQL.

### Lambda extension

Lambda function:

- internship-outbox-handler

Trigger:

- internship-prod-outbox SQS queue

Lambda responsibilities:

1. Consume SQS events.
2. Check event idempotency in DynamoDB.
3. Archive event JSON into S3.
4. Send email notification through Amazon SES.
5. Return partial batch failures for failed SQS messages.

DynamoDB deduplication table:

- InternshipLambdaEventDedupe

Successful smoke-test event:

- eventId: lambda-smoke-fixed-1785220478
- status: processed successfully
- result: EMAIL_SENT
- archive key:
  outbox-archive/2026/07/28/lambda-smoke-fixed-1785220478.json

### IAM and deployment

Important IAM roles:

- internship-github-deploy
- internship-eks-runtime
- internship-sagemaker-execution
- AmazonEKSLoadBalancerControllerRole
- internship-eks-node-role
- internship-lambda-outbox-role

GitHub Actions uses OIDC to assume the deployment role.

No long-lived AWS access key should be described as part of the deployment workflow.

## 4. GitHub Actions deployment modes

The workflow supports separate deployment modes:

- validate
- deploy-app
- deploy-public
- deploy-frontend
- rollout
- full

Expected responsibilities:

- validate:
  run tests, lint and security checks
- deploy-app:
  build Docker images, push ECR, deploy EKS workloads
- deploy-public:
  apply Ingress and reconcile ALB
- deploy-frontend:
  build React frontend, upload to S3, invalidate CloudFront
- rollout:
  restart existing Kubernetes deployments without building images
- full:
  perform the complete pipeline

## 5. Important architecture decisions

1. Frontend was removed from EKS and moved to S3 + CloudFront.
2. Backend and chat remain on EKS because they are long-running services.
3. Processing worker remains on EKS because AI tasks can be long-running.
4. Lambda is used for short event-driven work such as notification and audit.
5. Transactional outbox prevents business data from being committed without a corresponding event.
6. SQS decouples event producers and consumers.
7. DynamoDB conditional writes make Lambda event processing idempotent.
8. Redis provides realtime pub/sub but is not the permanent message database.
9. DynamoDB is the permanent storage layer for chat messages.
10. GitHub OIDC avoids storing long-lived AWS credentials.

## 6. Important troubleshooting history

### EKS node group failure

Problem:

- private subnet default route pointed to a deleted NAT Gateway
- route state was blackhole
- EKS nodes could not access required services and failed to join the cluster

Resolution:

- created or restored a NAT Gateway
- updated private subnet default routes
- recreated the managed node group
- nodes became Ready

### AWS Load Balancer Controller failure

Problem:

- controller could not determine VPC ID through instance metadata
- error:
  failed to get VPC ID from instance metadata

Resolution:

Added controller parameters:

- --aws-region=ap-southeast-1
- --aws-vpc-id=vpc-0cfee519122ae18b4

### Frontend deployment issue

Problem:

- frontend deployment job was skipped
- GitHub environment variables were not available inside a job-level `if`

Resolution:

- duplicated required deployment flags as repository variables
- configured VITE_API_BASE_URL as `/api`
- deployed frontend to S3
- created CloudFront S3 origin and behaviors

### Lambda DynamoDB error

Problem:

- DynamoDB UpdateExpression used the reserved attribute name `result`

Error:

- Invalid UpdateExpression
- reserved keyword: result

Resolution:

Changed:

- result = :result

To:

- #result = :result

And added:

- "#result": "result"

to ExpressionAttributeNames.

The next smoke-test event was processed successfully.