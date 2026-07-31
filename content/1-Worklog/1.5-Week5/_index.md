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

### Technical Implementation:

During Week 5, I developed an AI service with FastAPI to analyze CVs and Job Descriptions, extract structured information, and support candidate matching.

<pre>
Frontend / Backend
        |
        v
AI Service - FastAPI
        |
        +-- Text Cleaning
        +-- Prompt Processing
        +-- JSON Extraction
        +-- Validation
        +-- Matching and Scoring
        +-- Reranking
        |
        v
Structured AI Results
</pre>

### CV and Job Description Processing:

The AI service receives raw CV or Job Description content, cleans unnecessary text, sends the processed content to the model, and converts the model response into structured JSON.

Validation is applied before the result is returned to the backend.

<pre>
Raw Document Content
        |
        v
Clean and Normalize Text
        |
        v
AI Prompt
        |
        v
Extract JSON Response
        |
        v
Validate Structured Data
</pre>

The structured output can include fields such as skills, experience, education, job requirements, and preferred qualifications.

### Candidate Matching and Reranking:

Candidate matching compares structured CV information with Job Description requirements.

An initial score is calculated from relevant skills, experience, and other matching criteria. The Qwen reranker is then used to improve the final candidate order.

<pre>
Candidate Profiles
        |
        v
Initial Matching Score
        |
        v
Qwen Reranker
        |
        v
Ranked Candidate List
</pre>

The results are returned to the backend and displayed in the frontend for HR review.

### Asynchronous Processing:

Long-running AI operations were separated from normal API requests so that users would not need to wait for the entire analysis process.

The backend creates a processing job, while a worker handles the AI task in the background and stores the final result.

<pre>
User Request
    |
    v
Backend API
    |
    +-- Create processing job
    +-- Return job status
    |
    v
Queue / Worker
    |
    +-- Claim job
    +-- Run AI processing
    +-- Save result
    +-- Update status
</pre>

Amazon SQS was studied as a managed message queue for delivering jobs, while AWS Lambda was studied as an event-driven processing option.

### Retry and Idempotency:

Retry with backoff was applied to temporary failures. Instead of retrying continuously, the worker waits longer after each failed attempt.

Idempotency keys help prevent repeated requests from creating duplicate jobs or duplicate results.

<pre>
Processing Job
    |
    +-- Success
    |      |
    |      v
    |   Mark completed
    |
    +-- Temporary failure
    |      |
    |      v
    |   Retry with backoff
    |
    +-- Maximum attempts reached
           |
           v
        Mark failed
</pre>

### Concurrency and Transactional Outbox:

Optimistic concurrency control was used to reduce conflicts when multiple operations updated the same data.

The transactional outbox pattern keeps the business update and event creation in the same PostgreSQL transaction.

<pre>
Backend Transaction
        |
        +-- Update business data
        +-- Insert outbox event
        |
        v
Commit Transaction
        |
        v
Dispatcher publishes event
</pre>

If the transaction fails, neither the business change nor the outbox event is committed. If publishing fails later, the stored outbox event can be retried.

### Monitoring:

Amazon CloudWatch was studied for monitoring AI processing time, worker failures, retry counts, logs, and job status.

<pre>
AI Service and Worker
        |
        +-- Processing Logs
        +-- Error Logs
        +-- Job Duration
        +-- Retry Count
        |
        v
Amazon CloudWatch
</pre>

### Problems and Solutions:

| Problem | Resolution | Status |
|---|---|---|
| AI responses were not always structured consistently. | Added JSON extraction and schema validation. | Completed |
| Raw CV and Job Description text contained unnecessary content. | Added cleaning and normalization modules. | Completed |
| Initial matching scores were not sufficient for final ranking. | Added Qwen-based reranking. | Completed |
| AI processing could block the main API request. | Moved long-running work to processing jobs and workers. | Completed |
| Temporary failures could stop job execution. | Added retry with backoff. | Completed |
| Repeated requests could create duplicate jobs. | Added idempotency keys. | Completed |
| Concurrent updates could overwrite newer data. | Added optimistic concurrency control. | Completed |
| Events could be lost after a database commit. | Added the transactional outbox pattern. | Completed |

### Technical Knowledge Gained:

This week helped me understand how AI output should be cleaned, validated, and converted into structured application data.

I also learned that long-running AI tasks should be processed asynchronously instead of inside normal API requests.

Amazon SQS, AWS Lambda, and CloudWatch showed how AWS services can support event-driven processing, retry handling, and monitoring.

### Weekly Results:

By the end of Week 5, CVs and Job Descriptions could be analyzed into structured data and used for candidate matching and reranking.

The system also supported background processing, retry with backoff, idempotency, concurrency control, and transactional outbox handling.

### Lessons Learned:

AI integration requires more than sending prompts to a model. The system must also validate responses, manage failures, prevent duplicate processing, and monitor long-running jobs.

Asynchronous processing improves API responsiveness and makes AI workloads easier to retry and scale.

### Next Week Plan:

The next week will focus on containerizing the services, preparing Kubernetes deployment files, and improving automated testing, observability, and deployment reliability.

<!--
TODO: Add AI parsing results, matching screenshots, worker logs, retry tests, SQS or Lambda study evidence, CloudWatch monitoring, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-5/
-->