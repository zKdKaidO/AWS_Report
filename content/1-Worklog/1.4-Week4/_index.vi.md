---
title: "Nhật ký tuần 4"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Tuần 4 - Realtime chat với Socket.IO, Redis và DynamoDB

### Mục tiêu tuần 4:

- Xây dựng realtime communication giữa Candidate và HR.
- Lưu trữ chat data trong DynamoDB.
- Sử dụng Redis để hỗ trợ nhiều chat-service instances.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Khởi tạo chat service bằng Node.js, Express và Socket.IO. | 29/06/2026 | 05/07/2026 | [Socket.IO Documentation](https://socket.io/docs/v4/) |
| 2 | Xây dựng routes, controllers và Socket.IO events. | 29/06/2026 | 05/07/2026 | [Socket.IO Documentation](https://socket.io/docs/v4/) |
| 3 | Thiết kế ChatUsers, ChatGroups và ChatMessages trong DynamoDB. | 29/06/2026 | 05/07/2026 | [Amazon DynamoDB - Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html); [AWS SDK for JavaScript - DynamoDB Examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_dynamodb_code_examples.html) |
| 4 | Tích hợp Redis Socket.IO adapter. | 29/06/2026 | 05/07/2026 | [Socket.IO Redis Adapter Documentation](https://socket.io/docs/v4/redis-adapter/); [Redis Pub/Sub Documentation](https://redis.io/docs/latest/develop/pubsub/) |
| 5 | Tích hợp frontend Chat và chạy CORS/concurrency tests. | 29/06/2026 | 05/07/2026 | [Socket.IO Redis Adapter Documentation](https://socket.io/docs/v4/redis-adapter/); [Redis Pub/Sub Documentation](https://redis.io/docs/latest/develop/pubsub/) |

### Kết quả đạt được trong tuần:

- Candidate và HR có thể sử dụng realtime chat.
- Messages và groups được lưu trong DynamoDB.
- Redis đồng bộ events giữa nhiều instances.
- Chat service cung cấp health và metrics endpoints.
- CORS và concurrency behavior được kiểm thử.

<!--
Evidence required: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-4/
-->
