---
title: "Week 6 Worklog"
date: 2026-07-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Week 6 - System Containerization and Docker Compose Integration

### Week 6 Objectives:

- Standardize the runtime environment for the backend, frontend, chat service, and AI service.
- Integrate the main system components within a shared Docker Compose environment.
- Complete a repeatable startup process, service health checks, and demo data preparation.
- Prepare reusable instructions for running the full system locally.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Created and refined Dockerfiles for the backend, frontend, chat service, and AI service; standardized environment variables, working directories, dependency installation, and startup commands for each container. | 13/07/2026 | 13/07/2026 | [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/); [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/) |
| 2 | Configured PostgreSQL, Redis, and DynamoDB Local in Docker Compose; integrated the backend, worker, frontend, and chat service using internal networking, volumes, and service dependencies. | 15/07/2026 | 16/07/2026 | [Docker Compose Application Model](https://docs.docker.com/compose/intro/compose-application-model/); [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/); [Control Startup Order](https://docs.docker.com/compose/how-tos/startup-order/) |
| 3 | Separated the AI service into an independent Compose file, executed database migrations and demo data seeding, and performed health checks, smoke tests, and troubleshooting documentation for common startup issues. | 17/07/2026 | 17/07/2026 | [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/); [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/); [Control Startup Order](https://docs.docker.com/compose/how-tos/startup-order/) |

### Technical Implementation:

During Week 6, I containerized the backend, frontend, chat service, and AI service so that each component could run in a consistent and isolated environment.

<pre>
Frontend
Backend
Chat Service
AI Service
PostgreSQL
Redis
DynamoDB Local
        |
        v
Docker Compose Network
</pre>

Each service received its own Dockerfile with a defined working directory, dependency installation process, environment configuration, exposed port, and startup command.

### Docker Compose Integration:

Docker Compose was used to connect the main services in one local environment.

The backend communicates with PostgreSQL, while the chat service uses Redis and DynamoDB Local. Containers communicate through internal service names instead of fixed local IP addresses.

<pre>
Frontend
    |
    v
Backend
    |
    +-- PostgreSQL
    +-- Worker

Chat Service
    |
    +-- Redis
    +-- DynamoDB Local
</pre>

Volumes were added where persistent local data was required, while service dependencies and health checks helped control startup order.

### Service Startup and Health Checks:

A repeatable startup flow was prepared so that the full system could be launched with fewer manual steps.

<pre>
Build container images
        |
        v
Start infrastructure services
        |
        v
Run database migrations
        |
        v
Seed demo data
        |
        v
Start application services
        |
        v
Run health checks and smoke tests
</pre>

Health endpoints were used to confirm that services were running and able to connect to their required dependencies.

### AI Service Separation:

The AI service was placed in a separate Compose configuration because it may require different dependencies, larger resources, or a different startup workflow from the main application.

This separation allows the core system to run without always starting the AI workload.

### Problems and Solutions:

| Problem | Resolution | Status |
|---|---|---|
| Services behaved differently across developer machines. | Standardized runtimes using Dockerfiles. | Completed |
| Local setup required many manual commands. | Combined services through Docker Compose. | Completed |
| Containers could not use localhost to reach each other. | Used Compose service names and internal networking. | Completed |
| Backend startup could fail before PostgreSQL was ready. | Added service dependencies and health checks. | Completed |
| Local data disappeared after container recreation. | Added volumes for required persistent data. | Completed |
| Database structure and demo data were not initialized consistently. | Standardized migration and seeding steps. | Completed |
| AI dependencies made the main Compose environment heavier. | Separated the AI service into an independent Compose file. | Completed |

### Technical Knowledge Gained:

This week helped me understand how containers provide consistent runtime environments across different machines.

I also learned how Docker Compose manages service networking, environment variables, volumes, startup dependencies, and health checks.

Another important lesson was that a working container is not enough; the complete system also needs repeatable migrations, demo data, smoke tests, and troubleshooting instructions.

### Weekly Results:

By the end of Week 6, the main system components could run locally through Docker Compose.

The backend was connected to PostgreSQL, while the chat service communicated with Redis and DynamoDB Local. Health checks, database migrations, demo data seeding, and smoke tests were included in the startup process.

### Lessons Learned:

Containerization reduces environment differences, but service startup still requires clear dependency management and configuration.

Docker Compose makes local integration easier by placing services on a shared network and allowing them to communicate through service names.

### Next Week Plan:

The next week will focus on preparing Kubernetes resources, deploying containerized services, and improving configuration, secrets, scaling, and observability.

<!--
TODO: Add Docker container screenshots, Compose logs, health-check results, smoke-test evidence, or startup documentation for this week.
Expected image directory:
static/images/worklog/week-6/
-->