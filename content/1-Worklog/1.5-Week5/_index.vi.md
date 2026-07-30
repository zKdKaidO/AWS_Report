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

### Kết quả đạt được trong tuần:

- CV và Job Description có thể được làm sạch, phân tích và chuyển thành dữ liệu có cấu trúc.
- Hệ thống hỗ trợ candidate matching, scoring và reranking.
- Kết quả AI được tích hợp với backend và hiển thị trên giao diện frontend.
- Các tác vụ AI chạy lâu được tách khỏi request chính thông qua processing job và worker.
- Retry và idempotency giúp hạn chế lỗi tạm thời và tránh tạo dữ liệu trùng lặp.
- Concurrent updates và outbox events được xử lý ổn định hơn.
- Hiểu vai trò của Amazon SQS trong hàng đợi công việc, AWS Lambda trong xử lý sự kiện và CloudWatch trong theo dõi log, metrics và lỗi hệ thống.

<!--
TODO: Add AI parsing results, matching screenshots, worker logs, retry tests, SQS or Lambda study evidence, CloudWatch monitoring, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-5/
-->