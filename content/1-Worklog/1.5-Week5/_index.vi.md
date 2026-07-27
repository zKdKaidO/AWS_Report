---
title: "Nhật ký tuần 5"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Tuần 5 - Tích hợp AI, asynchronous processing và concurrency

### Mục tiêu tuần 5:

- Xây dựng AI service để phân tích CV và Job Description.
- Hỗ trợ candidate matching, scoring và reranking.
- Tách long-running jobs khỏi API requests và cải thiện độ tin cậy.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Khởi tạo AI service, health endpoints và request/response models. | 06/07/2026 | 12/07/2026 | [FastAPI Documentation - Request and Response Models](https://fastapi.tiangolo.com/tutorial/response-model/); [Amazon SageMaker AI - Real-Time Inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints.html) |
| 2 | Xây dựng cleaning, prompts và parsers cho Job Description và CV content. | 06/07/2026 | 12/07/2026 | [Hugging Face Transformers - Text Generation](https://huggingface.co/docs/transformers/en/llm_tutorial); [Prompt Engineering Guidelines](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |
| 3 | Xây dựng scoring, matching, Qwen reranker và tích hợp AI với backend/frontend. | 06/07/2026 | 12/07/2026 | [Hugging Face Transformers - Text Generation](https://huggingface.co/docs/transformers/en/llm_tutorial); [Amazon SageMaker AI - Real-Time Inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints.html) |
| 4 | Thiết kế processing jobs, worker lease, retry behavior và idempotency key support. | 06/07/2026 | 12/07/2026 | [AWS Prescriptive Guidance - Retry with Backoff Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html); [AWS Prescriptive Guidance - Asynchronous Communication](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-integrating-microservices/asynchronous.html) |
| 5 | Áp dụng optimistic concurrency control và transactional outbox dispatching. | 06/07/2026 | 12/07/2026 | [PostgreSQL Documentation - Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html); [AWS Prescriptive Guidance - Transactional Outbox Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html) |

### Kết quả đạt được trong tuần:

- CV và Job Description content có thể được chuyển thành structured data.
- Hệ thống hỗ trợ scoring, matching và reranking.
- AI và document processing không còn chặn main request.
- Repeated requests không tạo dữ liệu trùng lặp.
- Concurrent updates và outbox events được xử lý tin cậy hơn.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-5/
-->
