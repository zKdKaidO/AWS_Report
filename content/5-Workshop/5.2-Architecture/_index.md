---
title: "Architecture"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Objective

This chapter explains the final AWS architecture and the main runtime flows of the Internship Application Tracker.

## Scope

This architecture is based on the final production evidence, the project documentation notes, and the current application repository. I treat runtime evidence as the source of truth when it differs from older manifests. The most important current correction is that the frontend is hosted by S3 and CloudFront, not by EKS.

## Architecture context

| Layer | Main components |
|---|---|
| Edge | CloudFront distribution `EQIGYNECXDYL8` |
| Public routing | Application Load Balancer `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` |
| Kubernetes | EKS cluster `internship-prod`, namespace `internship` |
| Workloads | `backend`, `chat-service`, `backend-outbox-dispatcher`, `backend-processing-worker`, `ai-service` |
| Data | RDS PostgreSQL, DynamoDB, ElastiCache Redis, S3 |
| Async and event | PostgreSQL processing jobs, PostgreSQL outbox, SQS, Lambda, SES |
| AI | `ai-service` adapter and SageMaker endpoint `internship-qwen3-4b` |
| CI/CD | GitHub Actions, AWS OIDC, ECR, kubectl, AWS CLI |

## Diagram 1: Overall AWS architecture

{{< mermaid >}}
graph LR
    User["Candidate / HR browser"]
    CF["CloudFront (dhm2rz5nmsibj.cloudfront.net)"]
    S3Frontend["Private S3 frontend bucket (internship-prod-frontend-account-redacted)"]
    ALB["Application Load Balancer (internet-facing)"]
    EKS["EKS cluster (internship-prod)"]
    Backend["backend (FastAPI :8000)"]
    Chat["chat-service (Socket.IO :3000)"]
    Dispatcher["backend-outbox-dispatcher"]
    Worker["backend-processing-worker"]
    AI["ai-service (:8010)"]
    RDS["RDS PostgreSQL (internship-prod-postgres)"]
    DDB["DynamoDB chat tables (ChatUsers / ChatGroups / ChatMessages)"]
    Redis["ElastiCache Redis (internship-prod-redis)"]
    Uploads["S3 uploads and archive bucket (internship-prod-uploads-account-redacted)"]
    SQS["SQS queue (internship-prod-outbox)"]
    DLQ["SQS DLQ (internship-prod-outbox-dlq)"]
    Lambda["Lambda (internship-outbox-handler)"]
    Dedupe["DynamoDB dedupe table (InternshipLambdaEventDedupe)"]
    SES["Amazon SES"]
    SM["SageMaker endpoint (internship-qwen3-4b)"]

    User --> CF
    CF --> S3Frontend
    CF --> ALB
    ALB --> EKS
    EKS --> Backend
    EKS --> Chat
    EKS --> Dispatcher
    EKS --> Worker
    EKS --> AI
    Backend --> RDS
    Backend --> Uploads
    Chat --> DDB
    Chat --> Redis
    Worker --> AI
    AI --> SM
    Dispatcher --> SQS
    SQS --> Lambda
    SQS --> DLQ
    Lambda --> Dedupe
    Lambda --> Uploads
    Lambda --> SES
{{< /mermaid >}}

## Diagram 2: CloudFront request-routing flow

{{< mermaid >}}
graph TD
    Request["HTTPS request to CloudFront"]
    Match{"Path pattern"}
    Frontend["Default behavior to S3 frontend origin"]
    API["/api/* behavior to ALB origin"]
    ChatRest["/chat/* behavior to ALB origin"]
    Socket["/socket.io/* behavior to ALB origin"]
    BackendSvc["Kubernetes Service: backend"]
    ChatSvc["Kubernetes Service: chat-service"]

    Request --> Match
    Match --> Frontend
    Match --> API
    Match --> ChatRest
    Match --> Socket
    API --> BackendSvc
    ChatRest --> ChatSvc
    Socket --> ChatSvc
{{< /mermaid >}}

The frontend bucket remains private. CloudFront handles HTTPS and SPA fallback. The ALB Ingress rewrites `/api` and `/chat` prefixes before forwarding to the backend and chat services.

## Diagram 3: CV upload and AI processing sequence

{{< mermaid >}}
sequenceDiagram
    participant Candidate
    participant Frontend
    participant Backend
    participant S3 as S3 uploads bucket
    participant Postgres as RDS PostgreSQL
    participant Worker as Processing worker
    participant AI as ai-service
    participant SM as SageMaker

    Candidate->>Frontend: Submit application with CV
    Frontend->>Backend: POST /jobs/{id}/apply
    Backend->>S3: Store uploaded document or generate storage reference
    Backend->>Postgres: Commit application, idempotency record, outbox event, processing job
    Backend-->>Frontend: Application accepted with processing status
    Worker->>Postgres: Claim queued async_processing_jobs row
    Worker->>AI: POST /parse-cv or /match-applications
    AI->>SM: Invoke endpoint internship-qwen3-4b
    SM-->>AI: Model response
    AI-->>Worker: Normalized JSON response
    Worker->>Postgres: Save parsed profile or matching results
    Frontend->>Backend: Poll processing job status
    Backend-->>Frontend: Processing result
{{< /mermaid >}}

## Diagram 4: Realtime chat sequence

{{< mermaid >}}
sequenceDiagram
    participant UserA as User A browser
    participant CF as CloudFront
    participant ALB as ALB
    participant Chat1 as chat-service pod A
    participant Redis as ElastiCache Redis
    participant Chat2 as chat-service pod B
    participant DDB as DynamoDB chat tables
    participant UserB as User B browser

    UserA->>CF: Socket.IO connect /socket.io
    CF->>ALB: Forward websocket/polling traffic
    ALB->>Chat1: Route to chat-service
    UserA->>Chat1: message:send
    Chat1->>DDB: Persist message
    Chat1->>Redis: Publish room event
    Redis->>Chat2: Broadcast event to subscribed pod
    Chat2->>UserB: Emit message:new
    Chat1-->>UserA: ACK after persistence
{{< /mermaid >}}

DynamoDB is the permanent chat store. Redis is only the cross-pod pub/sub layer and does not replace persistent storage.

## Diagram 5: Transactional outbox, SQS, and Lambda sequence

{{< mermaid >}}
sequenceDiagram
    participant API as FastAPI backend
    participant PG as RDS PostgreSQL
    participant Dispatcher as Outbox dispatcher
    participant SQS as SQS main queue
    participant Lambda as Lambda outbox handler
    participant DDB as DynamoDB dedupe
    participant S3 as S3 archive bucket
    participant SES as Amazon SES

    API->>PG: Commit business mutation and outbox_events row in one transaction
    Dispatcher->>PG: Claim pending outbox_events row with lease
    Dispatcher->>SQS: Send event envelope
    Dispatcher->>PG: Mark event PUBLISHED
    SQS->>Lambda: Invoke batch
    Lambda->>DDB: Conditional write eventId
    alt First time event
        Lambda->>S3: Put JSON archive
        Lambda->>SES: Send email notification
    else Duplicate event
        Lambda-->>SQS: Treat as already processed
    end
{{< /mermaid >}}

SQS delivery is at least once. Duplicate delivery is expected, so Lambda idempotency is required. The successful smoke-test event was `lambda-smoke-fixed-1785220478`, archived under `outbox-archive/2026/07/28/lambda-smoke-fixed-1785220478.json`.

## Diagram 6: Source-code change and CI/CD deployment flow

{{< mermaid >}}
graph LR
    Dev["Developer change"]
    GH["GitHub Actions (workflow_dispatch mode)"]
    Validate["validate (tests, lint, build, infra scan)"]
    Images["build-images (ECR SHA tags)"]
    Restore["restore-compute (EKS node group check)"]
    App["deploy-app (kubectl apply workloads)"]
    Public["deploy-public (ALB ingress)"]
    CF["ensure-cloudfront (S3 private + OAC + behaviors)"]
    Frontend["deploy-frontend (npm build + S3 sync + invalidation)"]
    Summary["Production summary"]

    Dev --> GH
    GH --> Validate
    GH --> Restore
    Validate --> Images
    Restore --> Images
    Images --> App
    App --> Public
    Public --> CF
    CF --> Frontend
    App --> Summary
    Frontend --> Summary
{{< /mermaid >}}

The workflow supports `validate`, `deploy`, `rollout`, `restore-compute`, `deploy-app`, `deploy-public`, `deploy-frontend`, and `full`. GitHub Actions assumes an AWS role through OIDC; long-lived AWS access keys are not part of the deployment design.

## Component descriptions

| Component | Responsibility | Verification command |
|---|---|---|
| CloudFront | Public HTTPS entry point and routing | `aws cloudfront get-distribution --id EQIGYNECXDYL8` |
| S3 frontend bucket | Static frontend assets | `aws s3 ls s3://internship-prod-frontend-<AWS_ACCOUNT_ID>/` |
| ALB | Routes API/chat/socket traffic to EKS services | `kubectl get ingress internship-public -n internship` |
| Backend | FastAPI business API and health endpoints | `kubectl rollout status deployment/backend -n internship` |
| Chat service | Socket.IO and chat REST API | `kubectl rollout status deployment/chat-service -n internship` |
| Processing worker | Async AI/document jobs | `kubectl get deployment backend-processing-worker -n internship` |
| Outbox dispatcher | Publishes PostgreSQL events to SQS | `kubectl logs deployment/backend-outbox-dispatcher -n internship` |
| AI service | SageMaker adapter with stable worker routes | `kubectl port-forward service/ai-service 8010:8010 -n internship` |
| RDS PostgreSQL | Transactional data and queues | `aws rds describe-db-instances --db-instance-identifier internship-prod-postgres --region ap-southeast-1` |
| DynamoDB | Chat and Lambda dedupe tables | `aws dynamodb describe-table --table-name ChatMessages --region ap-southeast-1` |
| Redis | Socket.IO pub/sub | `aws elasticache describe-replication-groups --replication-group-id internship-prod-redis --region ap-southeast-1` |
| SQS | Outbox event transport | `aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All` |

## Public and private resources

| Public-facing | Private or internal |
|---|---|
| CloudFront domain | RDS PostgreSQL |
| ALB DNS origin | ElastiCache Redis |
| CloudFront HTTPS routes | DynamoDB tables |
| Browser-accessible static assets through CloudFront | EKS pod-to-service traffic |
| SES email delivery | SQS queue and DLQ |
|  | S3 buckets without direct public bucket access |

## Security boundaries

- GitHub Actions uses OIDC to assume `internship-github-deploy`.
- EKS workloads use `internship-app` ServiceAccount and IRSA role mapping.
- Secrets are injected through GitHub environment secrets and Kubernetes `internship-secrets`.
- S3 frontend bucket should remain private and readable through CloudFront only.
- Backend and chat pods expose only service ports through the ALB path rules.
- AI traffic stays inside the cluster from worker to `ai-service`; SageMaker is invoked by the adapter.
- SQS/Lambda processing is idempotent through DynamoDB conditional writes.

## Expected result

The final architecture separates static delivery, public request routing, long-running service execution, transactional storage, realtime synchronization, asynchronous event delivery, short event-driven notification, and AI inference. Each component has a clear responsibility and an independent verification path.

## Common errors

| Error | Cause | Resolution |
|---|---|---|
| Frontend shown as running in EKS | Old architecture description | Use current S3 and CloudFront architecture only |
| S3 described behind ALB | Incorrect CloudFront origin model | Default CloudFront behavior must route directly to S3 |
| Redis described as chat database | Misunderstood chat design | DynamoDB stores chat records; Redis is pub/sub |
| Outbox described as exactly-once | SQS Standard semantics | Document at-least-once plus idempotent consumer |
| Worker enabled before AI readiness | SageMaker dependency not ready | Keep worker disabled until AI and endpoint are ready |

## Outcome

This architecture chapter provides the reference model for the remaining workshop steps. All later deployment, testing, monitoring, security, cost, troubleshooting, and cleanup sections should match these diagrams.
