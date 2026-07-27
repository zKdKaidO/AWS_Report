---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Week 5 - AI Integration, Asynchronous Processing, and Concurrency

### Week 5 Objectives:

- Build the AI service for CV and Job Description analysis.
- Support candidate matching, scoring, and reranking.
- Separate long-running jobs from API requests and improve reliability.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Initialize the AI service, health endpoints, and request/response models. | 06/07/2026 | 12/07/2026 | [FastAPI Documentation - Request and Response Models](https://fastapi.tiangolo.com/tutorial/response-model/); [Amazon SageMaker AI - Real-Time Inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints.html) |
| 2 | Build cleaning, prompts, and parsers for Job Description and CV content. | 06/07/2026 | 12/07/2026 | [Hugging Face Transformers - Text Generation](https://huggingface.co/docs/transformers/en/llm_tutorial); [Prompt Engineering Guidelines](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |
| 3 | Build scoring, matching, Qwen reranker, and backend/frontend AI integration. | 06/07/2026 | 12/07/2026 | [Hugging Face Transformers - Text Generation](https://huggingface.co/docs/transformers/en/llm_tutorial); [Amazon SageMaker AI - Real-Time Inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints.html) |
| 4 | Design processing jobs, worker lease, retry behavior, and idempotency key support. | 06/07/2026 | 12/07/2026 | [AWS Prescriptive Guidance - Retry with Backoff Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html); [AWS Prescriptive Guidance - Asynchronous Communication](https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-integrating-microservices/asynchronous.html) |
| 5 | Apply optimistic concurrency control and transactional outbox dispatching. | 06/07/2026 | 12/07/2026 | [PostgreSQL Documentation - Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html); [AWS Prescriptive Guidance - Transactional Outbox Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html) |

### Week 5 Achievements:

- CV and Job Description content can be converted into structured data.
- The system supports scoring, matching, and reranking.
- AI and document processing no longer block the main request.
- Repeated requests do not create duplicate data.
- Concurrent updates and outbox events are handled more reliably.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-5/
-->
