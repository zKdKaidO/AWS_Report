---
title: "Nhật ký tuần 5"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Tuần 5 - Tích hợp AI, xử lý bất đồng bộ và tìm hiểu AWS Lambda, SQS

### Mục tiêu tuần 5:

- Xây dựng AI service để phân tích CV và Job Description.
- Chuyển nội dung CV và Job Description thành dữ liệu có cấu trúc.
- Phát triển candidate matching, scoring và reranking.
- Tách các tác vụ AI chạy lâu khỏi luồng xử lý chính của backend.
- Tìm hiểu AWS Lambda, Amazon SQS và Amazon CloudWatch trong kiến trúc xử lý bất đồng bộ.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Khởi tạo AI service bằng FastAPI, xây dựng health endpoint, request/response models và các module làm sạch nội dung CV, Job Description. | 06/07/2026 | 06/07/2026 | [FastAPI Response Models](https://fastapi.tiangolo.com/tutorial/response-model/); [Hugging Face Transformers](https://huggingface.co/docs/transformers/en/llm_tutorial) |
| 2 | Xây dựng prompt, JSON extraction và validation cho CV và Job Description; phát triển scoring, matching và Qwen reranker, sau đó tích hợp kết quả với backend và frontend. | 07/07/2026 | 08/07/2026 | [Hugging Face Transformers](https://huggingface.co/docs/transformers/en/llm_tutorial); [Prompt Engineering Guidelines](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |
| 3 | Tìm hiểu mô hình xử lý bất đồng bộ bằng Amazon SQS và AWS Lambda; thiết kế processing job, worker, retry with backoff và idempotency key để các tác vụ AI không chặn API request chính. | 09/07/2026 | 09/07/2026 | [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html); [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html); [Retry with Backoff Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html) |
| 4 | Hoàn thiện optimistic concurrency control và transactional outbox; kiểm thử repeated requests, concurrent updates và tìm hiểu cách theo dõi AI jobs, errors và processing time bằng Amazon CloudWatch. | 10/07/2026 | 10/07/2026 | [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html); [Transactional Outbox Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html); [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |

### Nội dung kỹ thuật đã triển khai:

Trong Tuần 5, tôi phát triển AI service bằng FastAPI để phân tích CV và Job Description, trích xuất dữ liệu có cấu trúc và hỗ trợ xếp hạng ứng viên.

<pre>
Frontend / Backend
        |
        v
AI Service - FastAPI
        |
        +-- Làm sạch văn bản
        +-- Xử lý prompt
        +-- Trích xuất JSON
        +-- Kiểm tra dữ liệu
        +-- Matching và Scoring
        +-- Reranking
        |
        v
Kết quả AI có cấu trúc
</pre>

### Xử lý CV và Job Description:

AI service nhận nội dung CV hoặc Job Description, làm sạch văn bản, gửi nội dung đến model và chuyển phản hồi thành JSON có cấu trúc.

Kết quả được kiểm tra trước khi trả về backend.

<pre>
Nội dung tài liệu
        |
        v
Làm sạch và chuẩn hóa
        |
        v
AI Prompt
        |
        v
Trích xuất JSON
        |
        v
Kiểm tra dữ liệu
</pre>

Dữ liệu đầu ra có thể bao gồm kỹ năng, kinh nghiệm, học vấn, yêu cầu công việc và các tiêu chí ưu tiên.

### Matching và Reranking ứng viên:

Quá trình matching so sánh dữ liệu CV với các yêu cầu trong Job Description.

Hệ thống tính điểm ban đầu dựa trên kỹ năng, kinh nghiệm và các tiêu chí liên quan. Sau đó, Qwen reranker được sử dụng để cải thiện thứ tự ứng viên.

<pre>
Danh sách ứng viên
        |
        v
Điểm matching ban đầu
        |
        v
Qwen Reranker
        |
        v
Danh sách đã xếp hạng
</pre>

Kết quả được gửi về backend và hiển thị trên frontend để HR xem xét.

### Xử lý bất đồng bộ:

Các tác vụ AI chạy lâu được tách khỏi API request chính để người dùng không phải chờ toàn bộ quá trình xử lý.

Backend tạo processing job, còn worker xử lý AI ở nền và lưu lại kết quả.

<pre>
User Request
    |
    v
Backend API
    |
    +-- Tạo processing job
    +-- Trả trạng thái job
    |
    v
Queue / Worker
    |
    +-- Nhận job
    +-- Chạy AI processing
    +-- Lưu kết quả
    +-- Cập nhật trạng thái
</pre>

Amazon SQS được nghiên cứu như một message queue được quản lý, còn AWS Lambda được nghiên cứu như một lựa chọn xử lý theo sự kiện.

### Retry và Idempotency:

Retry with backoff được áp dụng cho các lỗi tạm thời. Sau mỗi lần lỗi, worker chờ lâu hơn trước khi thử lại.

Idempotency key giúp ngăn request lặp tạo ra job hoặc kết quả trùng lặp.

<pre>
Processing Job
    |
    +-- Thành công
    |      |
    |      v
    |   Đánh dấu completed
    |
    +-- Lỗi tạm thời
    |      |
    |      v
    |   Retry with backoff
    |
    +-- Vượt số lần thử
           |
           v
       Đánh dấu failed
</pre>

### Concurrency và Transactional Outbox:

Optimistic concurrency control được sử dụng để hạn chế xung đột khi nhiều thao tác cùng cập nhật một dữ liệu.

Transactional outbox giữ business update và event trong cùng một PostgreSQL transaction.

<pre>
Backend Transaction
        |
        +-- Cập nhật dữ liệu
        +-- Thêm outbox event
        |
        v
Commit Transaction
        |
        v
Dispatcher publish event
</pre>

Nếu transaction thất bại, cả dữ liệu và event đều không được lưu. Nếu publish thất bại sau đó, outbox event vẫn có thể được retry.

### Monitoring:

Amazon CloudWatch được nghiên cứu để theo dõi thời gian xử lý AI, lỗi worker, số lần retry, log và trạng thái job.

<pre>
AI Service và Worker
        |
        +-- Processing Logs
        +-- Error Logs
        +-- Job Duration
        +-- Retry Count
        |
        v
Amazon CloudWatch
</pre>

### Vấn đề và cách giải quyết:

| Vấn đề | Cách giải quyết | Trạng thái |
|---|---|---|
| Phản hồi AI không luôn có cấu trúc ổn định. | Bổ sung JSON extraction và schema validation. | Hoàn thành |
| CV và Job Description chứa nhiều nội dung không cần thiết. | Bổ sung bước làm sạch và chuẩn hóa văn bản. | Hoàn thành |
| Điểm matching ban đầu chưa đủ tốt để xếp hạng cuối. | Bổ sung Qwen reranking. | Hoàn thành |
| AI processing có thể làm API request chờ lâu. | Chuyển tác vụ dài sang processing job và worker. | Hoàn thành |
| Lỗi tạm thời có thể làm job dừng hoàn toàn. | Bổ sung retry with backoff. | Hoàn thành |
| Request lặp có thể tạo job trùng. | Bổ sung idempotency key. | Hoàn thành |
| Nhiều cập nhật cùng lúc có thể ghi đè dữ liệu mới. | Bổ sung optimistic concurrency control. | Hoàn thành |
| Event có thể bị mất sau khi database commit. | Bổ sung transactional outbox. | Hoàn thành |

### Kiến thức kỹ thuật đã học:

Tuần này giúp tôi hiểu cách làm sạch, kiểm tra và chuyển phản hồi AI thành dữ liệu có cấu trúc.

Tôi cũng hiểu rằng các tác vụ AI chạy lâu nên được xử lý bất đồng bộ thay vì chạy trực tiếp trong API request.

Amazon SQS, AWS Lambda và CloudWatch cho thấy cách AWS hỗ trợ event-driven processing, retry và monitoring.

### Kết quả tuần:

Đến cuối Tuần 5, CV và Job Description có thể được phân tích thành dữ liệu có cấu trúc để phục vụ matching và reranking ứng viên.

Hệ thống cũng hỗ trợ background processing, retry with backoff, idempotency, concurrency control và transactional outbox.

### Bài học rút ra:

Tích hợp AI không chỉ là gửi prompt đến model mà còn cần validation, xử lý lỗi, chống xử lý trùng và monitoring.

Asynchronous processing giúp API phản hồi nhanh hơn và làm cho AI workload dễ retry, quản lý và mở rộng hơn.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung vào container hóa các service, chuẩn bị Kubernetes deployment, cải thiện automated testing, observability và độ ổn định khi triển khai.

<!--
TODO: Add AI parsing results, matching screenshots, worker logs, retry tests, SQS or Lambda study evidence, CloudWatch monitoring, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-5/
-->