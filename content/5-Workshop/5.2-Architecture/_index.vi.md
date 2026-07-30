---
title: "Kiến trúc"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Mục tiêu

Chương tài liệu này tường giải cấu trúc kiến trúc AWS sau cùng cũng như các luồng thực thi (runtime flows) trọng tâm của hệ thống Internship Application Tracker.

## Phạm vi

Kiến trúc này được phác thảo dựa trên minh chứng production sau cùng, tài liệu thiết kế ghi chú của dự án và kho lưu trữ mã nguồn hiện tại của ứng dụng. Tôi lấy bằng chứng vận hành khi chạy thực tế (runtime evidence) làm cội nguồn chân lý cho các tình huống mà nó có điểm xô lệch so với những manifest cũ. Điểm đính chính kiến trúc đáng chú ý nhất hiện nay là frontend được lưu giữ và phát tán qua bộ đôi S3 và CloudFront, không còn là workload Kubernetes.

## Bối cảnh kiến trúc

| Lớp (Layer) | Các thành phần chính (Main components) |
|---|---|
| Ngoại vi (Edge) | Tài nguyên CloudFront distribution `EQIGYNECXDYL8` |
| Định tuyến công cộng (Public routing) | Cống điều phối lưu lượng Application Load Balancer `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` |
| Cụm Kubernetes | Cụm máy EKS `internship-prod`, không gian tên `internship` |
| Khối workload ứng dụng | `backend`, `chat-service`, `backend-outbox-dispatcher`, `backend-processing-worker`, `ai-service` |
| Lớp dữ liệu (Data) | Cỗ máy RDS PostgreSQL, DynamoDB, ElastiCache Redis, kho object storage S3 |
| Lớp bất đồng bộ và sự kiện (Async and event) | Hàng đợi công việc trong PostgreSQL, bảng PostgreSQL outbox, hàng đợi SQS, hàm Lambda, dịch vụ phát thư SES |
| Trí tuệ nhân tạo (AI) | Pod trung gian `ai-service` adapter kết cọc cùng trạm suy đoán SageMaker endpoint `internship-qwen3-4b` |
| Dây chuyền CI/CD | GitHub Actions, ủy quyền AWS OIDC, kho ảnh ECR, công cụ kubectl, bộ lệnh AWS CLI |

## Sơ đồ 1: Tổng thể kiến trúc AWS

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

## Sơ đồ 2: Luồng định tuyến yêu cầu trên CloudFront

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

Kho lưu trữ bucket cho frontend duy trì an tọa trong tư thế khép kín (private). CDN CloudFront ôm gánh trách nhiệm mã hóa HTTPS và điều luật đường dự phòng cho giao diện trang đơn (SPA fallback). Luồng ALB Ingress thực thi cắt gõ xóa phần tiền tố `/api` cùng `/chat` (rewrites) trước thời khắc luân chuyển thẳng về cho 2 dịch vụ backend và chat.

## Sơ đồ 3: Biểu đồ tuần tự luồng tải CV và tác vụ xử lý AI

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

## Sơ đồ 4: Biểu đồ tuần tự luồng tin nhắn chat thời gian thực (Realtime chat)

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

Cơ sở dữ liệu DynamoDB là cõi lưu cất vững chắc vĩnh cửu cho tin nhắn chat. Cụm ElastiCache Redis duy chỉ làm nhiệm vụ trạm chuyển nhĩ cẩu pub/sub giao thoa giữa các pod và hoàn toàn không xô lấn thay thế thế trận lưu dữ liệu bền lâu (persistent storage).

## Sơ đồ 5: Biểu đồ tuần tự luồng transactional outbox, SQS và Lambda

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

Cơ quan vận chuyển tin SQS bám sát cam kết phân phối ít nhất một lần (at-least-once). Việc xuất hiện các nhịp tin chao bị nhận ban lặp lại là điều được tiên định trong thiết kế, do vậy khả năng xử lý bất biến (idempotency) của Lambda là nguyên tắc sống còn. Kịch bản test khói (smoke-test) ghi trát thành quả đắc thắng đã cất mã định danh sự kiện là `lambda-smoke-fixed-1785220478`, quy tập chôn an tọa dưới con rãnh S3 `outbox-archive/2026/07/28/lambda-smoke-fixed-1785220478.json`.

## Sơ đồ 6: Luồng triển khai CI/CD khi thay đổi mã nguồn

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

Dây chuyền kịch bản workflow linh hoạt cung cẩu dải chế độ thi hành gồm `validate`, `deploy`, `rollout`, `restore-compute`, `deploy-app`, `deploy-public`, `deploy-frontend`, và nhĩ lệnh cất còi trọn cõi `full`. Trình GitHub Actions thông quan xác nhận thông tính ủy quyền AWS role thông qua giao dịch OIDC; những chìa khóa bảo mật kiên bám AWS access keys dài hạn tuyệt đối không có vị trí nào trong quy hoạch thiết lập an sinh ban bố này.

## Mô tả các thành phần

| Thành phần (Component) | Trách nhiệm (Responsibility) | Lệnh kiểm tra |
|---|---|---|
| CloudFront | Cổng vào HTTPS công cộng cho người dùng đi đôi khả năng điều phối phân tuyến | `aws cloudfront get-distribution --id EQIGYNECXDYL8` |
| S3 frontend bucket | Kho cất các tệp tài sản tĩnh của frontend | `aws s3 ls s3://internship-prod-frontend-<AWS_ACCOUNT_ID>/` |
| ALB | Luân chuyển điều trút nhịp truy vãng API/chat/socket rẽ theo hướng vào các cụm service trên EKS | `kubectl get ingress internship-public -n internship` |
| Backend | Cánh cổng nghiệp vụ FastAPI kinh doanh kiêm cõi điểm gọi sức khỏe | `kubectl rollout status deployment/backend -n internship` |
| Dịch vụ chat (Chat service) | Xử trí giao thông realtime Socket.IO kết cọc cùng cõi chat REST API | `kubectl rollout status deployment/chat-service -n internship` |
| Processing worker | Gác nhiệm vụ rèn giải mác hồ sơ và cọc AI theo cơ cấu xử lý bất đồng bộ | `kubectl get deployment backend-processing-worker -n internship` |
| Outbox dispatcher | Chiết gặt nhĩ tệp sự kiện từ cơ quan PostgreSQL để bạt nạp sa cọc sang SQS | `kubectl logs deployment/backend-outbox-dispatcher -n internship` |
| Dịch vụ AI (AI service) | Pod trung gian (SageMaker adapter) buông đường vãng tuệ bền lâu cho đàn worker ra lời gọi | `kubectl port-forward service/ai-service 8010:8010 -n internship` |
| RDS PostgreSQL | Kho lưu giữ dữ liệu cho giao dịch và dải thông hàng đợi | `aws rds describe-db-instances --db-instance-identifier internship-prod-postgres --region ap-southeast-1` |
| DynamoDB | Nơi an trú các bảng chat cùng bàn lưu cất chứng tích khử lặp cho Lambda | `aws dynamodb describe-table --table-name ChatMessages --region ap-southeast-1` |
| Redis | Cơ quan chuyền tin phát chóp pub/sub mạn Socket.IO | `aws elasticache describe-replication-groups --replication-group-id internship-prod-redis --region ap-southeast-1` |
| SQS | Con hào cống vận tải cho dòng chảy tin outbox | `aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All` |

## Các tài nguyên công khai và riêng tư

| Hướng ra Internet (Public-facing) | Riêng tư hoặc nội bộ (Private or internal) |
|---|---|
| Tên miền công khai của CloudFront | Máy chủ CSDL RDS PostgreSQL |
| Tên miền DNS origin thuộc bãi ALB | Cụm máy nhân bản ElastiCache Redis |
| Chuỗi các ranh định tuyến HTTPS của CloudFront | Hệ thống các bàn dữ liệu bên DynamoDB |
| Các tài sản web tĩnh được trình duyệt tiếp thu qua CloudFront | Giao thông luân chuyển thẳng từ pod tới service cõi EKS |
| Khâu ném phát đi văn thư mail qua cống SES | Hàng đợi sự kiện chủ lực SQS đi đôi cọ bàn lỗi DLQ |
|  | Cõi hòm kho cất S3 bị đóng tịt, khước bẻ trần truy cập trực diện lộ thiên từ public |

## Ranh giới bảo mật (Security boundaries)

- Tiến trình GitHub Actions nương tuệ nhờ quyền OIDC để ra chiêu hoán quyền thâu cọc vào `internship-github-deploy`.
- Cụm các workload trên EKS vận dụng cấu hình ServiceAccount `internship-app` mang kèm theo bản gõ trói vai trò IRSA role mapping.
- Muôn thông ký mật mã bí ẩn thọ an tiêm truyền thẳng cẩu từ hòm bí mật của GitHub và cọc Kubernetes Secret mang mác `internship-secrets`.
- Kho chứa tệp tĩnh cho frontend trên S3 kỵ kiêng giêm giấu vùng kín riêng tư, duy nhĩ thọ ban visa mở cống qua duy nhất rào CDN CloudFront.
- Các pod của backend và dịch vụ chat thọ nhĩ bọc rẽ khe hở duy trì cọc cõi giao thiệp ứng với ranh luồn cổng service nhờ vào luật chia đường của ALB.
- Giao thông gọi dịch vụ AI trốn tiệm êm gọn trong ranh giới nội cluster đi từ pod worker tiến sang con pod `ai-service`; cõi trạm suy bạt SagMaker endpoint thọ khấn giục bục từ chính thợ bộ đệm adapter này.
- Nghiệp vụ giao đãi giữa SQS/Lambda bấu rào cản tính bất biến, thiêng cọc bảo lưu qua ranh lệnh chốt ghi theo điều kiện trên cỗi DynamoDB.

## Kết quả mong đợi

Cấu trúc kiến trúc trọn vẹn sau cùng thực hiện bóc rẽ trành minh tường chuỗi phát tán web tĩnh, định tuyến truy thu từ công cộng, các dịch vụ gác chạy mải khôn dừng, cõi cất trữ giao dịch có ACID, trạm đồng bộ tín liệu realtime, rỗng bão chuyển tải sự kiện bất đồng bộ, cọc cống tin tức chuế ngắn rêu cảnh báo, và cơ quan cất trát suy đoán AI. Trọn bộ từng thực thể sắm lấy một cương vị trách nhiệm tường minh cùng khả năng khảo rà, minh chứng hoạt vãng độc lập.

## Các lỗi thường gặp

| Lỗi (Error) | Nguyên nhân | Hướng khắc phục |
|---|---|---|
| Phác trát miêu tả frontend thọ chay nghênh gục bên giữa lòng EKS | Rơi nhầm vào văn bản phác tài liệu kiến trúc thảm xa xưa | Nhẫn tâm bấu 100% ranh giới mô hình frontend tải qua bộ đôi S3 và CloudFront CDN |
| Viết cẩu rằng kho S3 núp cẩn sau tấm khiên ALB | Lầm lọt mô hình thiết định origin cõi CloudFront | Behavior mặc định của CloudFront buộc bão dốc cầu trúng đích vào origin S3 thô trần |
| Xưng tụng nhầm cụm Redis chính là kho CSDL lưu chat vĩnh cửu | Hiểu sai thảm lật cấu trúc luồng lưu trữ đàm thoại chat | Bàn DynamoDB chịu cọc gánh việc ghi giữ tin nhắn trần; còn cõi Redis sắm quyền hạn bộ phát sóng pub/sub |
| Miêu tả cơ quan outbox là phân phối đúng nhĩ chính xác 1 lần (exactly-once) | Chưa am tường tập quy luật phát tin của SQS Standard | Văn khải mô tả rõ tiêu chuẩn gửi ít nhất một lần (at-least-once) cõng trói cùng thao thấu consumer có tính bất biến |
| Bồi cất kích bật công tắc worker trước lúc dàn hạ tầng AI sẵn sàng | Trụ cọc thọ sinh SagMaker chưa mở thông | Ép giữ worker an bế trong nhãn giam khóa tịt (disabled) đến mốc khoảnh khắc cụm AI cùng điểm endpoint thọ thông bãi |

## Kết luận

Chương trình Kiến trúc này cống hiến mẫu mẫn khuôn bản tham chiếu nòng cốt cho muôn chặng đường kiểm thử và thau dựng của Workshop kíp tiếp bước. Mọi cương mục sau đó chuyên về ban bố triển khai, nghiệm thu test, rà soát quan trắc, an nộ bảo mật, cước thọ ngân sách, xử trí lỗi lầm cùng thao tác gõ dẹp tháo dỡ thảy đều bám đuổi tuyệt đối trung thành theo nhĩ đàn sơ đồ tuần tự và hạ tầng đã tạc lên tại đây.
