---
title: "Week 4 Worklog"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Week 4 - Realtime Chat with Socket.IO, Redis, and DynamoDB

### Week 4 Objectives:

- Build realtime communication between Candidate and HR.
- Store chat data in DynamoDB.
- Use Redis to support multiple chat-service instances.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Initialize the chat service with Node.js, Express, and Socket.IO. | 29/06/2026 | 05/07/2026 | [Socket.IO Documentation](https://socket.io/docs/v4/) |
| 2 | Build routes, controllers, and Socket.IO events. | 29/06/2026 | 05/07/2026 | [Socket.IO Documentation](https://socket.io/docs/v4/) |
| 3 | Design ChatUsers, ChatGroups, and ChatMessages in DynamoDB. | 29/06/2026 | 05/07/2026 | [Amazon DynamoDB - Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html); [AWS SDK for JavaScript - DynamoDB Examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_dynamodb_code_examples.html) |
| 4 | Integrate the Redis Socket.IO adapter. | 29/06/2026 | 05/07/2026 | [Socket.IO Redis Adapter Documentation](https://socket.io/docs/v4/redis-adapter/); [Redis Pub/Sub Documentation](https://redis.io/docs/latest/develop/pubsub/) |
| 5 | Integrate frontend Chat and run CORS/concurrency tests. | 29/06/2026 | 05/07/2026 | [Socket.IO Redis Adapter Documentation](https://socket.io/docs/v4/redis-adapter/); [Redis Pub/Sub Documentation](https://redis.io/docs/latest/develop/pubsub/) |

### Week 4 Achievements:

- Candidate and HR can use realtime chat.
- Messages and groups are stored in DynamoDB.
- Redis synchronizes events across multiple instances.
- The chat service provides health and metrics endpoints.
- CORS and concurrency behavior are tested.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-4/
-->
