---
title: "Kiến trúc"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
# Kiến trúc

## Mục tiêu

Chương này mô tả kiến trúc AWS cuối cùng và các luồng chạy chính của hệ thống Internship Application Tracker.

## Phạm vi

Kiến trúc được viết dựa trên ngữ cảnh production đã cung cấp, ghi chú trong repository Report và repository ứng dụng hiện tại. Bằng chứng runtime được ưu tiên hơn các manifest cũ. Điểm chỉnh quan trọng nhất là frontend hiện được phục vụ bằng S3 và CloudFront, không chạy như một workload trong EKS.

## Bối cảnh kiến trúc

| Lớp | Thành phần chính |
|---|---|
| Edge | CloudFront distribution `EQIGYNECXDYL8` |
| Public routing | Application Load Balancer `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` |
| Kubernetes | EKS cluster `internship-prod`, namespace `internship` |
| Workloads | `backend`, `chat-service`, `backend-outbox-dispatcher`, `backend-processing-worker`, `ai-service` |
| Dữ liệu | RDS PostgreSQL, DynamoDB, ElastiCache Redis, S3 |
| Bất đồng bộ và sự kiện | PostgreSQL processing jobs, PostgreSQL outbox, SQS, Lambda, SES |
| AI | Adapter `ai-service` và SageMaker endpoint `internship-qwen3-4b` |
| CI/CD | GitHub Actions, AWS OIDC, ECR, kubectl, AWS CLI |

## Diagram 1: Kiến trúc AWS tổng thể

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

## Diagram 2: Luồng định tuyến request qua CloudFront

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

Frontend bucket vẫn là private. CloudFront xử lý HTTPS và SPA fallback. ALB Ingress rewrite prefix `/api` và `/chat` trước khi chuyển tiếp đến backend và chat service.

## Diagram 3: Luồng upload CV và xử lý AI

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

## Diagram 4: Luồng realtime chat

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

DynamoDB là nơi lưu trữ chat lâu dài. Redis chỉ là lớp pub/sub giữa nhiều pod và không thay thế persistent storage.

## Diagram 5: Luồng transactional outbox, SQS và Lambda

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

SQS có cơ chế at-least-once delivery. Duplicate delivery là tình huống bình thường, vì vậy Lambda phải xử lý idempotency. Smoke test thành công có event `lambda-smoke-fixed-1785220478`, được archive tại `outbox-archive/2026/07/28/lambda-smoke-fixed-1785220478.json`.

## Diagram 6: Luồng thay đổi source code và CI/CD deployment

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

Workflow hỗ trợ các mode `validate`, `deploy`, `rollout`, `restore-compute`, `deploy-app`, `deploy-public`, `deploy-frontend` và `full`. GitHub Actions assume AWS role thông qua OIDC; long-lived AWS access keys không nằm trong thiết kế deployment.

## Mô tả thành phần

| Thành phần | Trách nhiệm | Lệnh kiểm tra |
|---|---|---|
| CloudFront | Public HTTPS entry point và định tuyến request | `aws cloudfront get-distribution --id EQIGYNECXDYL8` |
| S3 frontend bucket | Lưu static frontend assets | `aws s3 ls s3://internship-prod-frontend-<AWS_ACCOUNT_ID>/` |
| ALB | Định tuyến API/chat/socket traffic đến EKS services | `kubectl get ingress internship-public -n internship` |
| Backend | FastAPI business API và health endpoints | `kubectl rollout status deployment/backend -n internship` |
| Chat service | Socket.IO và chat REST API | `kubectl rollout status deployment/chat-service -n internship` |
| Processing worker | Async AI/document jobs | `kubectl get deployment backend-processing-worker -n internship` |
| Outbox dispatcher | Publish PostgreSQL events sang SQS | `kubectl logs deployment/backend-outbox-dispatcher -n internship` |
| AI service | SageMaker adapter với worker routes ổn định | `kubectl port-forward service/ai-service 8010:8010 -n internship` |
| RDS PostgreSQL | Transactional data và queues | `aws rds describe-db-instances --db-instance-identifier internship-prod-postgres --region ap-southeast-1` |
| DynamoDB | Chat tables và Lambda dedupe table | `aws dynamodb describe-table --table-name ChatMessages --region ap-southeast-1` |
| Redis | Socket.IO pub/sub | `aws elasticache describe-replication-groups --replication-group-id internship-prod-redis --region ap-southeast-1` |
| SQS | Outbox event transport | `aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All` |

## Tài nguyên public và private

| Public-facing | Private hoặc internal |
|---|---|
| CloudFront domain | RDS PostgreSQL |
| ALB DNS origin | ElastiCache Redis |
| CloudFront HTTPS routes | DynamoDB tables |
| Static assets truy cập qua CloudFront | EKS pod-to-service traffic |
| SES email delivery | SQS queue và DLQ |
|  | S3 buckets không mở public bucket access trực tiếp |

## Ranh giới bảo mật

- GitHub Actions dùng OIDC để assume role `internship-github-deploy`.
- EKS workloads dùng ServiceAccount `internship-app` và IRSA role mapping.
- Secrets được inject qua GitHub environment secrets và Kubernetes `internship-secrets`.
- S3 frontend bucket phải giữ private và chỉ cho CloudFront đọc.
- Backend và chat pods chỉ expose service ports qua ALB path rules.
- AI traffic đi nội bộ trong cluster từ worker đến `ai-service`; SageMaker được gọi bởi adapter.
- SQS/Lambda processing xử lý idempotency bằng DynamoDB conditional writes.

## Kết quả mong đợi

Kiến trúc cuối cùng tách rõ static delivery, public request routing, long-running service execution, transactional storage, realtime synchronization, asynchronous event delivery, short event-driven notification và AI inference. Mỗi thành phần có trách nhiệm rõ ràng và có đường kiểm tra độc lập.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| Mô tả frontend chạy trong EKS | Tài liệu kiến trúc cũ | Chỉ dùng kiến trúc hiện tại: S3 và CloudFront |
| Mô tả S3 nằm sau ALB | Hiểu sai origin model của CloudFront | Default CloudFront behavior phải route trực tiếp đến S3 |
| Mô tả Redis là chat database | Hiểu sai thiết kế chat | DynamoDB lưu chat records; Redis chỉ dùng pub/sub |
| Mô tả outbox là exactly-once | SQS Standard không đảm bảo exactly-once | Ghi rõ at-least-once và consumer idempotent |
| Enable worker trước khi AI sẵn sàng | SageMaker dependency chưa ready | Giữ worker disabled cho đến khi AI service và endpoint sẵn sàng |

## Kết luận

Chương Architecture này là mô hình tham chiếu cho các bước workshop còn lại. Các phần deployment, testing, monitoring, security, cost, troubleshooting và cleanup phía sau cần khớp với các diagram này.
