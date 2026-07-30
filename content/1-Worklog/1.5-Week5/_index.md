---
title: "Week 5 Worklog"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Week 5 - AI Integration, Asynchronous Processing, and AWS Lambda and SQS Fundamentals

### Week 5 Objectives:

- Build an AI service for analyzing CVs and Job Descriptions.
- Transform CV and Job Description content into structured data.
- Develop candidate matching, scoring, and reranking.
- Separate long-running AI tasks from the main backend request flow.
- Learn how AWS Lambda, Amazon SQS, and Amazon CloudWatch support asynchronous processing architectures.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Initialized the AI service with FastAPI and implemented health endpoints, request and response models, and content-cleaning modules for CVs and Job Descriptions. | 06/07/2026 | 06/07/2026 | [FastAPI Response Models](https://fastapi.tiangolo.com/tutorial/response-model/); [Hugging Face Transformers](https://huggingface.co/docs/transformers/en/llm_tutorial) |
| 2 | Developed prompts, JSON extraction, and validation for CV and Job Description data; implemented scoring, matching, and the Qwen reranker; and integrated the results with the backend and frontend. | 07/07/2026 | 08/07/2026 | [Hugging Face Transformers](https://huggingface.co/docs/transformers/en/llm_tutorial); [Prompt Engineering Guidelines](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |
| 3 | Studied asynchronous processing with Amazon SQS and AWS Lambda; designed processing jobs, workers, retry with backoff, and idempotency keys so that long-running AI tasks would not block the main API request. | 09/07/2026 | 09/07/2026 | [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html); [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html); [Retry with Backoff Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html) |
| 4 | Completed optimistic concurrency control and transactional outbox handling, tested repeated requests and concurrent updates, and studied monitoring of AI jobs, errors, and processing time with Amazon CloudWatch. | 10/07/2026 | 10/07/2026 | [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html); [Transactional Outbox Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html); [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |

### Week 5 Achievements:

- Enabled CV and Job Description content to be cleaned, analyzed, and converted into structured data.
- Implemented candidate matching, scoring, and reranking.
- Integrated AI results with the backend and frontend interfaces.
- Separated long-running AI tasks from the main request flow using processing jobs and workers.
- Used retry and idempotency mechanisms to reduce temporary failures and prevent duplicate data.
- Improved the reliability of concurrent updates and outbox event processing.
- Understood how Amazon SQS supports job queues, AWS Lambda supports event-driven processing, and CloudWatch supports log, metric, and error monitoring.

<!--
TODO: Add AI parsing results, matching screenshots, worker logs, retry tests, SQS or Lambda study evidence, CloudWatch monitoring, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-5/
-->