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

### Technical Implementation:

During Week 4, I developed a realtime chat service using Node.js, Express, and Socket.IO. The service allows Candidate and HR/Company users to join chat groups, send messages, receive new messages immediately, and reconnect after temporary disconnection.

<pre>
React Chat Interface
        |
        v
Socket.IO Client
        |
        v
Node.js Chat Service
        |
        +-- Authentication Middleware
        +-- Room Management
        +-- Message Events
        +-- Reconnection Handling
        |
        v
DynamoDB and Redis
</pre>

### Socket.IO Communication Flow:

After the user logs in, the frontend opens a Socket.IO connection using the current authentication state. The chat service validates the connection before allowing the user to join a room.

When a message is sent, the server validates the sender, stores the message, and broadcasts it to the relevant chat group.

<pre>
User sends message
        |
        v
Socket.IO event
        |
        v
Validate user and chat group
        |
        v
Store message in DynamoDB
        |
        v
Broadcast to group members
</pre>

The frontend also listens for disconnect and reconnect events so that the user can continue the conversation after a temporary network interruption.

### DynamoDB Data Design:

Amazon DynamoDB was used to design the storage model for chat users, chat groups, and chat messages.

The main tables were separated according to their responsibilities:

<pre>
ChatUsers
    |
    +-- User identity
    +-- Display information

ChatGroups
    |
    +-- Group information
    +-- Group membership

ChatMessages
    |
    +-- Group ID
    +-- Message timestamp
    +-- Sender
    +-- Message content
</pre>

Partition keys and sort keys were selected based on the main access patterns, especially loading a user’s chat groups and retrieving message history in chronological order.

### Redis Pub/Sub and Multiple Instances:

A single Socket.IO instance can deliver events to its own connected clients. However, when the chat service runs on multiple instances, users connected to different instances must still receive the same messages.

The Socket.IO Redis adapter was integrated to synchronize events between service instances.

<pre>
Chat Instance A
        |
        v
Redis Pub/Sub
        |
        v
Chat Instance B
        |
        v
Connected users receive event
</pre>

Amazon ElastiCache for Redis was studied as the managed AWS option for hosting Redis with improved availability, monitoring, and operational support.

### Frontend Chat Integration:

The React chat interface was connected to the Socket.IO client and the existing authentication state.

Users can select a chat group, load previous messages, send new messages, and receive realtime updates without refreshing the page.

The interface also includes basic connection-status handling so users can recognize when the realtime connection is interrupted or restored.

### Monitoring and Health Checks:

Basic health and metrics endpoints were added to support service monitoring.

The chat service can expose information such as service availability, active connections, and basic runtime status. Amazon CloudWatch was studied for collecting logs and monitoring service behavior after deployment.

<pre>
Chat Service
    |
    +-- Application Logs
    +-- Connection Metrics
    +-- Health Endpoint
    |
    v
Amazon CloudWatch
</pre>

### Problems and Solutions:

| Problem | Resolution | Status |
|---|---|---|
| Users needed immediate message delivery. | Implemented Socket.IO events for realtime communication. | Completed |
| Unauthorized users could attempt to open a socket connection. | Added authentication middleware before room access. | Completed |
| Messages needed persistent storage. | Designed ChatMessages storage in DynamoDB. | Completed |
| Multiple chat instances could not share events automatically. | Added the Socket.IO Redis adapter. | Completed |
| Temporary network loss interrupted the chat session. | Added disconnect and reconnect handling. | Completed |
| Realtime service behavior was difficult to observe. | Added basic health and metrics endpoints and studied CloudWatch monitoring. | Completed |

### Technical Knowledge Gained:

This week helped me understand the difference between REST communication and realtime event-based communication.

I also learned that DynamoDB design should begin with access patterns, while Redis Pub/Sub helps synchronize events between multiple service instances.

Another important lesson was that realtime systems require authentication, reconnection handling, message persistence, and monitoring, not only message delivery.

### Weekly Results:

By the end of Week 4, Candidate and HR/Company users could exchange messages in realtime through the React interface.

The chat service supported authenticated connections, room-based messaging, DynamoDB storage design, Redis-based event synchronization, reconnection behavior, and basic monitoring endpoints.

### Lessons Learned:

Realtime applications require coordination between the frontend, WebSocket service, persistent storage, and shared messaging infrastructure.

Socket.IO manages client communication, DynamoDB stores chat data, Redis synchronizes multiple service instances, and CloudWatch supports operational monitoring.

### Next Week Plan:

The next week will focus on packaging services with containers, preparing Kubernetes deployment resources, and improving health checks, configuration management, and deployment repeatability.

<!--
TODO: Add chat interface screenshots, Socket.IO event tests, DynamoDB table design, Redis adapter configuration, CloudWatch logs, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-4/
-->