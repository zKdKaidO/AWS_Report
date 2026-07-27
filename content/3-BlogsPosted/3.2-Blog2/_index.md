---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# BUILDING A SCALABLE REAL-TIME CHAT SYSTEM WITH SOCKET.IO, REDIS, AND DYNAMODB

Realtime chat appears in many systems such as e-commerce, recruitment platforms, customer support, social networks, and collaboration tools.

At a basic level, a single server can receive and send messages. However, when the number of users grows, the system needs to answer more difficult questions:

- How can messages appear immediately?
- What happens when the sender and receiver are connected to different servers?
- Where should messages be stored so they are not lost when a server restarts?
- How can the number of servers increase without interrupting users?

A common architecture for solving these problems combines Socket.IO, Redis, and Amazon DynamoDB.

## 1. Why REST API is not enough for realtime chat

REST API follows a request-response model:

```text
Client sends request
↓
Server processes
↓
Server returns response
```

If REST API is used for chat, the frontend usually needs to keep calling an API to check for new messages:

```text
GET /messages
Wait a few seconds
GET /messages
Wait a few seconds
GET /messages
```

This approach is called polling.

Polling is simple, but it has several limitations:

- Message latency depends on the polling interval.
- The client sends requests even when there are no new messages.
- The backend has to process many unnecessary requests.
- System load increases quickly as the number of users grows.

For systems that require near-instant responses, WebSocket or a realtime library such as Socket.IO is usually more suitable.

## 2. Socket.IO for realtime communication

Socket.IO allows a two-way connection to be maintained between the client and the server.

After the connection is established, the client and server can send events to each other without creating a new HTTP request for every exchange.

A simple message flow can be understood as:

```text
User A sends a message
↓
Socket.IO server receives the event
↓
Server processes the message
↓
Server sends a new event
↓
User B receives the message
```

Common events include:

- `conversation:join`
- `message:send`
- `message:new`
- `user:typing`
- `message:read`

Socket.IO also supports:

- Automatic reconnection when the network is interrupted.
- Grouping users into rooms.
- Sending events to one user or a group of users.
- Fallback when WebSocket is not available.

However, Socket.IO works best when the system has only one server. When the application runs multiple servers, an additional synchronization mechanism is required.

## 3. The issue when the chat service runs multiple instances

With one chat server, both users connect to the same server:

```text
User A ─┐
        ├── Chat server
User B ─┘
```

When User A sends a message, the server can send it directly to User B's connection.

As traffic grows, the system may need multiple servers:

```text
Chat server 1
Chat server 2
Chat server 3
```

A load balancer may distribute users like this:

```text
User A → Chat server 1
User B → Chat server 2
```

At this point, Chat server 1 does not directly manage User B's connection.

Without communication between servers, User A's message may be saved successfully, but User B may not receive a realtime notification.

This is the problem Redis can solve.

## 4. Redis synchronizes events between chat servers

Redis is a high-speed in-memory data store. In realtime chat architecture, Redis is commonly used for publish/subscribe.

The flow works like this:

```text
User A
↓
Chat server 1
↓ Publish
Redis
↓ Subscribe
Chat server 2
↓
User B
```

When Chat server 1 receives a message:

- The server processes and stores the message.
- The server publishes an event to Redis.
- Other chat servers receive the event.
- The server that holds the receiver's connection sends the message to the client.

Redis works like a shared broadcast channel between chat servers.

Redis is suitable for this role because:

- It has high processing speed.
- It supports publish/subscribe.
- It is suitable for short-lived data and events.
- It allows servers to operate independently while still exchanging events.

Redis does not have to be the long-term storage location for chat history. That task should be handled by a more durable database.

## 5. DynamoDB stores chat data

Amazon DynamoDB is a managed NoSQL database service from AWS.

Chat data often has these characteristics:

- The number of messages continuously increases.
- Write throughput can be high.
- Messages are usually queried by conversation.
- Complex joins are rarely needed.
- Usage can spike suddenly.

DynamoDB fits this model because it scales well and does not require users to manage database servers.

A simple message table design can use:

- Partition key: `conversation_id`
- Sort key: `timestamp` or `message_id`

In this design:

- `conversation_id` groups messages from the same conversation.
- `timestamp` sorts messages by time.

When chat history is needed, the system queries records with the same `conversation_id`.

The roles of Redis and DynamoDB can be separated clearly:

```text
Redis
→ Transmits realtime events between servers

DynamoDB
→ Stores long-term chat history
```

## 6. Complete message sending flow

A message can pass through the system in these steps:

1. The user enters a message.
2. The client sends an event through Socket.IO.
3. The chat server authenticates the user.
4. The server checks whether the user can access the conversation.
5. The message is stored in DynamoDB.
6. The server publishes an event through Redis.
7. The chat server holding the receiver's connection receives the event.
8. The server sends the new message to the receiver's client.

General architecture:

```text
Client A
│
│ Socket.IO
▼
Chat server 1
│
├──── Store data ─────► DynamoDB
│
└──── Publish event ──► Redis
                         │
                         ▼
                  Chat server 2
                         │
                         ▼
                      Client B
```

This architecture achieves two goals:

- Realtime responses through Socket.IO and Redis.
- Durable message history through DynamoDB.

## 7. How Kubernetes supports scaling chat services

When the application is packaged as containers, Kubernetes can run multiple replicas of the chat service.

```text
Chat service
├── Pod 1
├── Pod 2
├── Pod 3
└── Pod N
```

A Kubernetes Service or load balancer distributes connections to the pods.

Kubernetes also supports:

- Restarting containers automatically when failures occur.
- Rolling updates for new versions.
- Scaling the number of pods up or down.
- Health checks through liveness and readiness probes.
- Traffic distribution across multiple pods.
- Horizontal Pod Autoscaler based on CPU, memory, or custom metrics.

However, Kubernetes only runs and manages multiple instances. Realtime synchronization between instances still requires Redis or a suitable message broker.

## 8. Sticky session and Redis solve different problems

When Socket.IO uses HTTP long polling, a client can create multiple consecutive requests in the same session.

If these requests are routed to different servers, the connection session may fail.

Sticky session keeps requests from the same client on the same server:

```text
Client A → Chat server 1
Client A → Chat server 1
Client A → Chat server 1
```

However, sticky session does not replace Redis.

The difference is:

- Sticky session keeps one client's connection stable on one server.
- Redis transmits events between different servers.

Even when sticky session is enabled, the sender and receiver may still be connected to different servers.

## 9. Deployment on AWS

On AWS, the architecture can use:

```text
Application Load Balancer
↓
Amazon EKS or container service
↓
Multiple chat service instances
├── Amazon DynamoDB
└── Amazon ElastiCache/Valkey
```

Service roles:

- Amazon EKS: runs and manages chat service containers.
- Application Load Balancer: receives and distributes user connections.
- Amazon DynamoDB: stores chat data.
- Amazon ElastiCache or Valkey: provides a Redis-compatible service.
- Amazon CloudWatch: stores logs, metrics, and alarms.

For small projects or test environments, Redis and DynamoDB Local can be run with Docker before using real AWS services.

## 10. Common mistakes when building realtime chat

### Storing messages only in server memory

If the server restarts, all data may be lost.

```text
Server restart → Message history is lost
```

Messages should be stored in a durable database.

### Running multiple servers without a Redis adapter

Each server only knows the clients directly connected to it. Users on another server may not receive events.

### Using Redis as the only storage layer

Redis is suitable for cache and event transmission, but it is not always suitable as the only source of chat history.

### Not authenticating Socket.IO connections

WebSocket or Socket.IO connections still need to be authenticated with JWT, session, or another appropriate mechanism.

### Not checking conversation access

A logged-in user is not automatically allowed to join every chat room.

### Not handling duplicate messages

When the network is unstable, the client may resend the same message. A client-generated message ID or idempotency key can help reduce duplicate records.

### Not limiting message size and sending frequency

The system should include:

- Message length limits.
- Rate limiting.
- Spam prevention.
- File type validation if attachments are supported.

## 11. Cost optimization ideas

Realtime chat on AWS can generate cost from compute, load balancers, databases, Redis, and logs.

Some optimization ideas:

- Use DynamoDB On-Demand when traffic is low or unpredictable.
- Choose a suitable ElastiCache node instead of over-provisioning.
- Use Auto Scaling to avoid running too many instances during low traffic.
- Set retention periods for CloudWatch Logs instead of keeping logs forever.
- Use DynamoDB TTL for temporary data when the business rules allow it.
- Delete Load Balancers, ElastiCache clusters, or test environments after labs are completed.
- Run the system with Docker and DynamoDB Local during development.

Managed services reduce operational work, but resources still need to be monitored to avoid unexpected cost.

## 12. Reference checklist

- Use Socket.IO or WebSocket for realtime communication.
- Use a Redis adapter when running multiple chat servers.
- Store message history in a durable database.
- Design an appropriate DynamoDB partition key.
- Do not store important state only in server memory.
- Configure sticky session if using long polling.
- Authenticate realtime connections.
- Check conversation access permission.
- Add rate limiting and message size limits.
- Configure liveness and readiness probes.
- Monitor connection count, message rate, latency, and error rate.
- Configure Auto Scaling and resource limits properly.
- Check AWS resource cost after testing.

The most important lesson is that realtime chat is not only about setting up a WebSocket connection.

When the system starts to scale, three responsibilities should be separated clearly:

```text
Socket.IO
→ Realtime communication between client and server

Redis
→ Event synchronization between multiple servers

DynamoDB
→ Long-term chat data storage
```

Choosing the right tool for each responsibility helps the system respond quickly while remaining scalable as the number of users increases.

`#AWS` `#SocketIO` `#Redis` `#DynamoDB` `#AmazonEKS` `#RealtimeChat` `#NodeJS` `#Kubernetes` `#CloudComputing` `#AWSStudyGroup`
