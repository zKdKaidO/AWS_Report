---
title: "Đề xuất dự án"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


## Internship Application Platform

**Nền tảng quản lý ứng tuyển thực tập tích hợp AI và triển khai trên AWS**

## Tóm tắt dự án

Internship Application Platform là nền tảng hỗ trợ sinh viên, ứng viên và doanh nghiệp quản lý toàn bộ quá trình tuyển dụng thực tập trên một hệ thống tập trung.

Đối với Candidate, hệ thống hỗ trợ tạo hồ sơ cá nhân, tìm kiếm cơ hội thực tập, theo dõi trạng thái ứng tuyển, quản lý CV và các tài liệu liên quan, trao đổi trực tiếp với HR và sử dụng AI để đánh giá mức độ phù hợp giữa CV với Job Description.

Đối với HR và doanh nghiệp, hệ thống hỗ trợ quản lý thông tin công ty, đăng tin tuyển dụng, xem danh sách ứng viên, cập nhật trạng thái tuyển dụng, trao đổi với Candidate và sử dụng AI để phân tích, so sánh và xếp hạng hồ sơ.

Project được xây dựng theo kiến trúc nhiều dịch vụ, bao gồm:

- Frontend phát triển bằng React và Vite.
- Backend REST API phát triển bằng FastAPI.
- Chat service sử dụng Node.js, Socket.IO, Redis và DynamoDB.
- AI service hỗ trợ phân tích CV, Job Description và xếp hạng ứng viên.
- PostgreSQL lưu dữ liệu nghiệp vụ.
- Amazon S3 lưu CV và tài liệu.
- Docker và Kubernetes chuẩn hóa môi trường triển khai.
- Observability thu thập metrics, logs và distributed traces.
- GitHub Actions hỗ trợ CI/CD và kiểm tra bảo mật.

Kiến trúc production được định hướng triển khai trên AWS với Amazon EKS, Amazon ECR, Amazon RDS for PostgreSQL, Amazon DynamoDB, Amazon ElastiCache, Amazon S3, Amazon CloudFront, Application Load Balancer và các dịch vụ giám sát phù hợp.

## Phát biểu vấn đề

### Vấn đề hiện tại

Trong quá trình tìm kiếm thực tập, sinh viên thường quản lý thông tin ứng tuyển bằng bảng tính, ghi chú cá nhân, email hoặc nhiều nền tảng riêng lẻ. Cách làm này dẫn đến nhiều vấn đề:

- Thông tin công việc và doanh nghiệp bị phân tán.
- Khó theo dõi trạng thái của từng hồ sơ ứng tuyển.
- CV, bảng điểm, chứng chỉ và tài liệu liên quan được lưu ở nhiều vị trí.
- Candidate khó đánh giá CV của mình phù hợp với Job Description ở mức nào.
- Việc trao đổi với HR không được liên kết trực tiếp với hồ sơ ứng tuyển.
- Candidate dễ bỏ lỡ thời hạn, lịch phỏng vấn hoặc yêu cầu bổ sung tài liệu.

Đối với HR và doanh nghiệp, quá trình tuyển dụng thủ công cũng tạo ra nhiều hạn chế:

- Mất nhiều thời gian để đọc và phân loại CV.
- Khó so sánh nhiều ứng viên theo cùng một tiêu chí.
- Trạng thái tuyển dụng có thể không được cập nhật đồng nhất.
- Dữ liệu ứng viên có nguy cơ bị lưu trữ không an toàn.
- Việc trao đổi giữa HR và Candidate bị phân tán qua nhiều kênh.
- Thiếu công cụ tập trung để theo dõi hoạt động và hiệu quả tuyển dụng.

### Giải pháp đề xuất

Internship Application Platform cung cấp một hệ thống tập trung cho cả Candidate và HR.

Candidate có thể:

- Đăng ký và quản lý tài khoản.
- Cập nhật hồ sơ cá nhân.
- Tìm kiếm và xem thông tin công việc.
- Nộp hồ sơ ứng tuyển.
- Theo dõi trạng thái tuyển dụng.
- Upload và quản lý CV, chứng chỉ và bảng điểm.
- Trao đổi trực tiếp với HR.
- Sử dụng AI để phân tích CV và mức độ phù hợp với công việc.

HR và doanh nghiệp có thể:

- Tạo và quản lý hồ sơ công ty.
- Đăng và chỉnh sửa tin tuyển dụng.
- Xem danh sách hồ sơ ứng tuyển.
- Cập nhật trạng thái của ứng viên.
- Trao đổi trực tiếp với Candidate.
- Sử dụng AI để trích xuất thông tin, tính điểm và hỗ trợ xếp hạng ứng viên.

Các tác vụ mất nhiều thời gian như đọc tài liệu, phân tích CV và reranking được xử lý bất đồng bộ thông qua worker, giúp API chính không bị chặn trong thời gian xử lý.

## Lợi ích và giá trị

Đối với Candidate, nền tảng giúp:

- Quản lý toàn bộ quá trình ứng tuyển tại một nơi.
- Hạn chế thất lạc tài liệu và thông tin công việc.
- Theo dõi rõ trạng thái của từng application.
- Nhận biết kỹ năng phù hợp và kỹ năng còn thiếu.
- Trao đổi thuận tiện hơn với doanh nghiệp.

Đối với HR, nền tảng giúp:

- Giảm thời gian xử lý hồ sơ thủ công.
- Tập trung thông tin ứng viên và trạng thái tuyển dụng.
- Chuẩn hóa quá trình đánh giá hồ sơ.
- Hỗ trợ ra quyết định bằng dữ liệu.
- Cải thiện khả năng theo dõi và phối hợp tuyển dụng.

Đối với nhóm phát triển, project tạo cơ hội áp dụng kiến thức về full-stack development, Cloud architecture, AI integration, realtime communication, Kubernetes, CI/CD, observability và bảo mật hệ thống.

## Kiến trúc giải pháp

Hệ thống được thiết kế theo kiến trúc nhiều dịch vụ để tách biệt các nhóm chức năng, tăng khả năng bảo trì và hỗ trợ mở rộng độc lập.

```text
Candidate / HR
       |
       v
React Frontend
       |
       v
Amazon CloudFront
       |
       +-------------------------------+
       v                               v
Application Load Balancer       Amazon S3
       |                       Frontend assets
       v
Amazon EKS
+-- FastAPI Backend
+-- Processing Worker
+-- Outbox Dispatcher
+-- Chat Service
+-- AI Service / AI Client
       |
       +-- Amazon RDS PostgreSQL
       +-- Amazon DynamoDB
       +-- Amazon ElastiCache
       +-- Amazon S3
       +-- Amazon SQS
       +-- Amazon SageMaker

Metrics / Logs / Traces
       |
       v
CloudWatch / Prometheus / Grafana / Loki / Tempo
```

<!--
TODO: Add the real architecture diagram when available:
static/images/proposal/internship-platform-architecture.png
-->

### Các dịch vụ AWS được sử dụng

| Dịch vụ AWS | Vai trò trong hệ thống | Lý do lựa chọn |
|---|---|---|
| Amazon EKS | Chạy backend, worker, chat service và các workload dạng container | Hỗ trợ Kubernetes, scaling, rolling update và quản lý nhiều service |
| Amazon ECR | Lưu trữ container image | Tích hợp trực tiếp với EKS và GitHub Actions |
| Application Load Balancer | Tiếp nhận và phân phối request đến các service | Hỗ trợ routing, health check và tích hợp với Kubernetes Ingress |
| Amazon RDS for PostgreSQL | Lưu user, company, job, application, document metadata và processing job | Phù hợp với dữ liệu quan hệ và transaction |
| Amazon DynamoDB | Lưu user chat, conversation và message | Phù hợp với dữ liệu chat có lưu lượng đọc ghi lớn |
| Amazon ElastiCache | Redis pub/sub cho Socket.IO và hỗ trợ dữ liệu tạm thời | Giúp nhiều chat-service instance đồng bộ sự kiện |
| Amazon S3 | Lưu CV, chứng chỉ, bảng điểm và frontend assets | Có khả năng mở rộng, độ bền cao và hỗ trợ presigned URL |
| Amazon CloudFront | Phân phối frontend và static assets | Cải thiện tốc độ truy cập và hỗ trợ HTTPS |
| Amazon SQS | Nhận sự kiện bất đồng bộ từ outbox dispatcher | Giảm sự phụ thuộc trực tiếp giữa các service |
| Amazon SageMaker | Cung cấp endpoint suy luận AI khi triển khai production | Hỗ trợ quản lý model endpoint và khả năng mở rộng |
| Amazon CloudWatch | Thu thập log, metric và cảnh báo AWS | Hỗ trợ vận hành và điều tra sự cố |
| AWS IAM | Quản lý quyền truy cập giữa workload và AWS resource | Hỗ trợ nguyên tắc quyền tối thiểu |

### Thiết kế các thành phần

**Frontend**

Frontend được phát triển bằng React và Vite, cung cấp giao diện cho Candidate và HR. Static files có thể được build và lưu trên Amazon S3, sau đó phân phối qua Amazon CloudFront. Frontend giao tiếp với backend và chat service thông qua các endpoint được công bố qua Application Load Balancer.

**Backend API**

Backend được phát triển bằng FastAPI và chịu trách nhiệm xác thực người dùng, quản lý Candidate và HR, quản lý công ty và tin tuyển dụng, quản lý application và document metadata, tạo presigned URL, khởi tạo processing job, đồng thời cung cấp dashboard và dữ liệu nghiệp vụ.

**Processing Worker**

Processing worker thực hiện các tác vụ lâu như trích xuất nội dung CV, phân tích Job Description, phân tích CV, tính điểm matching và rerank ứng viên. Cơ chế lease, retry và giới hạn số lần thử được sử dụng để giảm lỗi khi worker bị dừng giữa quá trình xử lý.

**Chat Service**

Chat service sử dụng Node.js, Express và Socket.IO để hỗ trợ giao tiếp realtime. DynamoDB lưu user, group và message; Redis adapter đồng bộ sự kiện giữa nhiều pod; sticky session hỗ trợ duy trì kết nối Socket.IO khi có nhiều replica.

**AI Service**

AI service phân tích CV và Job Description, chuyển dữ liệu văn bản thành cấu trúc có thể xử lý và hỗ trợ tính điểm phù hợp. AI service cần có schema validation, timeout, retry có giới hạn, health check, kiểm soát log để không làm lộ dữ liệu CV và cơ chế fallback khi model không khả dụng.

**Data Storage**

PostgreSQL lưu dữ liệu có quan hệ và cần transaction. DynamoDB lưu dữ liệu chat. Amazon S3 lưu file lớn. Redis hỗ trợ realtime event distribution. Việc lựa chọn nhiều loại lưu trữ giúp mỗi nhóm dữ liệu được quản lý bằng công nghệ phù hợp với đặc điểm truy cập.

**Observability**

Hệ thống thu thập metrics để theo dõi hiệu năng và trạng thái, logs để kiểm tra sự kiện và lỗi, distributed traces để theo dõi request qua nhiều service. Prometheus, Grafana, Loki, OpenTelemetry và Tempo được sử dụng trong môi trường Kubernetes; Amazon CloudWatch được sử dụng để theo dõi resource và log trong môi trường AWS.

## Triển khai kỹ thuật

### Các giai đoạn triển khai

| Giai đoạn | Nội dung chính |
|---|---|
| 1. Phân tích và thiết kế | Phân tích bài toán Candidate-HR, xác định yêu cầu chức năng và phi chức năng, thiết kế database, thiết kế kiến trúc nhiều dịch vụ và xác định dịch vụ AWS cần sử dụng |
| 2. Xây dựng nền tảng nghiệp vụ | Xây dựng authentication, Candidate/HR profile, company management, job management, application tracking và document management |
| 3. Tích hợp realtime và AI | Xây dựng chat service, tích hợp Socket.IO, lưu chat data bằng DynamoDB, tích hợp Redis adapter, xây dựng AI service, phân tích CV/JD, matching và reranking |
| 4. Nâng cao độ tin cậy | Xây dựng processing worker, retry, lease, idempotency, optimistic concurrency, transactional outbox, validation và error handling |
| 5. Container hóa và Kubernetes | Viết Dockerfile, xây dựng Docker Compose, tạo local Kubernetes cluster bằng kind, triển khai Deployment, Service, Ingress, HPA, PDB, health probe và migration Job |
| 6. Observability và CI/CD | Thu thập metrics, logs, traces, xây dựng dashboard, thiết lập alert, xây dựng GitHub Actions, chạy automated test, smoke test và quét bảo mật |
| 7. Triển khai AWS và hoàn thiện | Build image, push lên Amazon ECR, triển khai workload lên Amazon EKS, kết nối RDS, DynamoDB, ElastiCache, S3, triển khai frontend lên S3/CloudFront, kiểm thử end-to-end và hoàn thiện báo cáo |

### Yêu cầu kỹ thuật

| Thành phần | Công nghệ hoặc yêu cầu |
|---|---|
| Frontend | React, Vite, Tailwind CSS |
| Backend | Python, FastAPI, SQLAlchemy, Alembic |
| Chat service | Node.js, Express, Socket.IO |
| AI service | Python, FastAPI, Qwen-compatible model hoặc SageMaker endpoint |
| Relational database | PostgreSQL |
| Chat database | DynamoDB |
| Cache/pub-sub | Redis hoặc Amazon ElastiCache |
| Object storage | Amazon S3 |
| Containers | Docker |
| Container orchestration | Kubernetes, kind và Amazon EKS |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus, Grafana và CloudWatch |
| Logs | Loki và CloudWatch Logs |
| Tracing | OpenTelemetry và Tempo |
| Security | JWT, IAM Role, OIDC, least privilege và secret management |

## Tiến độ và cột mốc

Kế hoạch được điều chỉnh theo 8 tuần, từ 08/06/2026 đến 30/07/2026, để thống nhất với Worklog của báo cáo.

| Tuần | Thời gian | Cột mốc | Kết quả dự kiến |
|---|---|---|---|
| 1 | 08/06/2026 - 14/06/2026 | Phân tích yêu cầu, kiến trúc, backend foundation và authentication | Hoàn thành phạm vi, kiến trúc tổng thể, đăng ký, đăng nhập và migration ban đầu |
| 2 | 15/06/2026 - 21/06/2026 | Candidate-HR platform, applications, documents và S3 | Hoàn thành company, jobs, application flow, upload và tải tài liệu an toàn |
| 3 | 22/06/2026 - 28/06/2026 | Frontend và REST API integration | Hoàn thành giao diện Candidate/HR, protected routes và API integration |
| 4 | 29/06/2026 - 05/07/2026 | Realtime chat | Candidate và HR trao đổi realtime qua Socket.IO, Redis và DynamoDB |
| 5 | 06/07/2026 - 12/07/2026 | AI integration và reliable processing | Phân tích CV/JD, matching, worker, idempotency, concurrency và outbox |
| 6 | 13/07/2026 - 19/07/2026 | Docker Compose | Chạy được full stack trong local và có quy trình smoke test |
| 7 | 20/07/2026 - 26/07/2026 | Kubernetes và observability | Chạy hệ thống trên kind, có metrics, logs, traces và alert |
| 8 | 27/07/2026 - 30/07/2026 | CI/CD, AWS và báo cáo | Kiểm thử end-to-end, chuẩn bị deployment AWS và hoàn thiện tài liệu |

## Ước tính ngân sách

Chi phí thực tế phụ thuộc vào Region, loại instance, thời gian chạy, dữ liệu lưu trữ, lưu lượng mạng và số lượng request. Không nên điền số tiền cố định trước khi cấu hình các resource thực tế trong AWS Pricing Calculator.

| Dịch vụ | Cấu hình dự kiến | Chi phí ước tính mỗi tháng |
|---|---|---:|
| Amazon EKS | Một cluster | TBD |
| EC2 worker nodes | Instance type và số node thực tế | TBD |
| Amazon ECR | Dung lượng container image | TBD |
| Amazon RDS PostgreSQL | Instance class, storage và backup | TBD |
| Amazon DynamoDB | On-demand hoặc provisioned capacity | TBD |
| Amazon ElastiCache | Node type và số node | TBD |
| Amazon S3 | CV, tài liệu và frontend assets | TBD |
| Amazon CloudFront | Data transfer và request | TBD |
| Application Load Balancer | Số giờ và lưu lượng | TBD |
| Amazon SQS | Số lượng message | TBD |
| Amazon SageMaker | Loại endpoint và thời gian hoạt động | TBD |
| Amazon CloudWatch | Log ingestion, retention và metric | TBD |
| Data transfer | Lưu lượng outbound | TBD |
| Tổng cộng |  | TBD |

### Giải pháp tối ưu chi phí

- Chỉ chạy môi trường production khi cần demo hoặc kiểm thử.
- Sử dụng instance phù hợp với tải thực tế.
- Giới hạn số worker node.
- Tắt hoặc xóa SageMaker endpoint khi không sử dụng.
- Thiết lập retention hợp lý cho log.
- Xóa image cũ trong ECR.
- Áp dụng S3 lifecycle policy khi phù hợp.
- Sử dụng DynamoDB on-demand trong giai đoạn có lưu lượng chưa ổn định.
- Cấu hình AWS Budget và Billing Alarm.
- Xóa Load Balancer, public IPv4, snapshot và volume không sử dụng.
- Tránh tạo NAT Gateway nếu kiến trúc workshop không thật sự cần.

## Đánh giá rủi ro

### Ma trận rủi ro

| Rủi ro | Khả năng | Ảnh hưởng | Mức độ |
|---|---|---|---|
| Lộ secret hoặc AWS credential | Thấp | Rất cao | Cao |
| RDS hoặc service bị public không cần thiết | Trung bình | Cao | Cao |
| CV và dữ liệu cá nhân bị truy cập trái phép | Thấp | Rất cao | Cao |
| AI trả kết quả không chính xác | Trung bình | Trung bình | Trung bình |
| AI service hết tài nguyên hoặc timeout | Trung bình | Cao | Cao |
| Chat realtime mất kết nối khi scale | Trung bình | Trung bình | Trung bình |
| Worker xử lý trùng job | Trung bình | Cao | Cao |
| Database update xảy ra race condition | Trung bình | Cao | Cao |
| Kubernetes pod hoặc node bị lỗi | Trung bình | Trung bình | Trung bình |
| Chi phí AWS vượt dự kiến | Trung bình | Cao | Cao |
| CI/CD deployment thất bại | Trung bình | Trung bình | Trung bình |
| Third-party package có lỗ hổng | Trung bình | Cao | Cao |

### Biện pháp giảm thiểu

**Bảo mật**

- Sử dụng IAM Role và nguyên tắc quyền tối thiểu.
- Không lưu AWS Access Key trong source code.
- Sử dụng GitHub Actions OIDC.
- Đưa `.env`, private key và secret ra khỏi repository.
- Giữ S3 bucket ở trạng thái private.
- Sử dụng presigned URL có thời hạn.
- Không ghi toàn bộ nội dung CV hoặc token vào log.
- Kiểm tra authorization và object ownership trên backend.

**Độ tin cậy**

- Sử dụng readiness và liveness probes.
- Dùng HPA để hỗ trợ scaling.
- Dùng PDB để giảm gián đoạn khi bảo trì.
- Sử dụng retry có giới hạn.
- Áp dụng processing-job lease.
- Sử dụng idempotency key.
- Áp dụng optimistic concurrency.
- Dùng transactional outbox cho event.

**AI**

- Kiểm tra output bằng schema.
- Thiết lập timeout.
- Giới hạn retry.
- Cho phép human review trước quyết định tuyển dụng.
- Không sử dụng điểm AI như quyết định tuyển dụng duy nhất.
- Ghi nhận lý do hoặc các yếu tố góp phần vào kết quả khi có thể.

**Chi phí**

- Thiết lập AWS Budget.
- Theo dõi Cost Explorer.
- Xóa resource ngay sau workshop.
- Giới hạn thời gian hoạt động của AI endpoint.
- Sử dụng log retention phù hợp.
- Kiểm tra EBS, Elastic IP, Load Balancer và snapshot còn sót.

### Kế hoạch dự phòng

- Nếu EKS chưa sẵn sàng, sử dụng Docker Compose hoặc local Kubernetes để demo.
- Nếu AI model không hoạt động, sử dụng mock response hoặc deterministic scoring để tiếp tục kiểm tra business flow.
- Nếu Redis không sẵn sàng, chạy một chat-service instance cho môi trường demo.
- Nếu S3 không truy cập được, sử dụng local storage trong development.
- Nếu worker gặp lỗi, processing job được retry theo lease và attempt limit.
- Nếu deployment mới thất bại, rollback về container image hoặc Kubernetes revision trước đó.
- Nếu chi phí vượt giới hạn, dừng workload không thiết yếu và xóa resource có phí cao.

## Kết quả mong đợi

Sau khi hoàn thành project, hệ thống dự kiến đạt được các kết quả:

- Cung cấp nền tảng tập trung cho Candidate và HR.
- Quản lý đầy đủ company, jobs, applications và documents.
- Lưu CV và tài liệu bằng cơ chế an toàn.
- Hỗ trợ chat realtime.
- Hỗ trợ AI phân tích CV và Job Description.
- Xử lý tác vụ lâu bằng worker.
- Kiểm soát request lặp và cập nhật đồng thời.
- Chạy được bằng Docker Compose và Kubernetes.
- Có CI/CD pipeline.
- Có metrics, logs, traces và alerts.
- Có kiến trúc triển khai trên AWS.
- Có tài liệu hướng dẫn triển khai, kiểm thử và clean-up.

### Giá trị lâu dài

Project có thể tiếp tục phát triển thành một hệ thống hoàn chỉnh với các chức năng:

- Gửi email hoặc notification khi trạng thái application thay đổi.
- Đặt lịch phỏng vấn.
- Tạo cover letter bằng AI.
- Sinh câu hỏi phỏng vấn dựa trên CV và Job Description.
- Đề xuất công việc phù hợp với hồ sơ Candidate.
- Dashboard phân tích hiệu quả tuyển dụng.
- Tích hợp calendar và email.
- Cải thiện explainability cho AI matching.
- Xây dựng mobile application.
- Mở rộng cho nhiều trường đại học và doanh nghiệp.

Ngoài giá trị sản phẩm, project còn cung cấp một tài liệu tham khảo về cách xây dựng hệ thống Cloud-native nhiều dịch vụ có tích hợp AI, realtime communication, asynchronous processing, Kubernetes, observability và CI/CD.
