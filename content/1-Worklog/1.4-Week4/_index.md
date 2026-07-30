---
title: "Week 4 Worklog"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Week 4 - Realtime Chat with Socket.IO, DynamoDB, and Redis

### Week 4 Objectives:

- Build realtime communication between Candidate and HR/Company users.
- Design the storage of users, chat groups, and messages with Amazon DynamoDB.
- Learn Redis Pub/Sub and Amazon ElastiCache for Redis for supporting multiple chat-service instances.
- Integrate the chat interface with the frontend and test concurrent connections.
- Become familiar with monitoring service logs and metrics using Amazon CloudWatch.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Initialized the chat service with Node.js, Express, and Socket.IO; implemented the WebSocket connection, authentication middleware, and basic events for sending and receiving messages. | 29/06/2026 | 29/06/2026 | [Socket.IO Documentation](https://socket.io/docs/v4/); [Express Documentation](https://expressjs.com/) |
| 2 | Designed the ChatUsers, ChatGroups, and ChatMessages tables in Amazon DynamoDB; defined partition keys, sort keys, and the main queries required for group lists and message history. | 30/06/2026 | 01/07/2026 | [Amazon DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html); [AWS SDK for JavaScript - DynamoDB Examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_dynamodb_code_examples.html) |
| 3 | Integrated the Redis adapter for Socket.IO to synchronize events across multiple chat-service instances; studied Redis Pub/Sub and how Amazon ElastiCache can provide managed Redis on AWS. | 02/07/2026 | 02/07/2026 | [Socket.IO Redis Adapter](https://socket.io/docs/v4/redis-adapter/); [Redis Pub/Sub Documentation](https://redis.io/docs/latest/develop/pubsub/); [Amazon ElastiCache for Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html) |
| 4 | Integrated the Chat interface into the React frontend, tested CORS, reconnection, and concurrent connections, added basic health and metrics endpoints, and studied log monitoring with Amazon CloudWatch. | 03/07/2026 | 03/07/2026 | [Socket.IO Client API](https://socket.io/docs/v4/client-api/); [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS); [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |

### Week 4 Achievements:

- Enabled Candidate and HR/Company users to create chat groups and exchange messages in real time.
- Implemented connection, message delivery, message reception, and reconnection events in the chat service.
- Completed the storage design for ChatUsers, ChatGroups, and ChatMessages in Amazon DynamoDB.
- Used the Redis adapter to synchronize Socket.IO events across multiple service instances.
- Integrated the chat interface with the authentication state and backend chat service.
- Tested CORS behavior, disconnected clients, and concurrent user connections.
- Understood the roles of Amazon ElastiCache for operating Redis and Amazon CloudWatch for monitoring logs, metrics, and service health.

<!--
TODO: Add chat interface screenshots, Socket.IO event tests, DynamoDB table design, Redis adapter configuration, CloudWatch logs, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-4/
-->