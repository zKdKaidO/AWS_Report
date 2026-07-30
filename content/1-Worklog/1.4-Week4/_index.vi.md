---
title: "Nhật ký tuần 4"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Tuần 4 - Realtime chat với Socket.IO, DynamoDB và Redis

### Mục tiêu tuần 4:

- Xây dựng chức năng realtime chat giữa Candidate và HR/Company.
- Thiết kế cách lưu trữ người dùng, nhóm trò chuyện và tin nhắn bằng Amazon DynamoDB.
- Tìm hiểu Redis Pub/Sub và Amazon ElastiCache for Redis để hỗ trợ nhiều chat-service instances.
- Tích hợp giao diện chat với frontend và kiểm tra các trường hợp kết nối đồng thời.
- Làm quen với việc theo dõi log và metrics của dịch vụ bằng Amazon CloudWatch.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Khởi tạo chat service bằng Node.js, Express và Socket.IO; xây dựng kết nối WebSocket, authentication middleware và các event cơ bản cho việc gửi, nhận tin nhắn. | 29/06/2026 | 29/06/2026 | [Socket.IO Documentation](https://socket.io/docs/v4/); [Express Documentation](https://expressjs.com/) |
| 2 | Thiết kế các bảng ChatUsers, ChatGroups và ChatMessages trong Amazon DynamoDB; xác định partition key, sort key và các truy vấn cần thiết cho danh sách nhóm và lịch sử tin nhắn. | 30/06/2026 | 01/07/2026 | [Amazon DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html); [AWS SDK for JavaScript - DynamoDB Examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_dynamodb_code_examples.html) |
| 3 | Tích hợp Redis adapter cho Socket.IO để đồng bộ sự kiện giữa nhiều chat-service instances; tìm hiểu Redis Pub/Sub và cách Amazon ElastiCache có thể cung cấp Redis được quản lý trên AWS. | 02/07/2026 | 02/07/2026 | [Socket.IO Redis Adapter](https://socket.io/docs/v4/redis-adapter/); [Redis Pub/Sub Documentation](https://redis.io/docs/latest/develop/pubsub/); [Amazon ElastiCache for Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html) |
| 4 | Tích hợp giao diện Chat vào React frontend, kiểm tra CORS, reconnect và concurrent connections; bổ sung health endpoint, metrics cơ bản và tìm hiểu cách theo dõi log bằng Amazon CloudWatch. | 03/07/2026 | 03/07/2026 | [Socket.IO Client API](https://socket.io/docs/v4/client-api/); [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS); [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |

### Kết quả đạt được trong tuần:

- Candidate và HR/Company có thể tạo nhóm trò chuyện và trao đổi tin nhắn theo thời gian thực.
- Các sự kiện kết nối, gửi tin nhắn, nhận tin nhắn và reconnect đã được xử lý trong chat service.
- Hoàn thiện thiết kế lưu trữ ChatUsers, ChatGroups và ChatMessages trên Amazon DynamoDB.
- Redis adapter hỗ trợ đồng bộ Socket.IO events giữa nhiều service instances.
- Giao diện chat được tích hợp với authentication state và backend chat service.
- Các trường hợp CORS, mất kết nối và nhiều người dùng truy cập đồng thời đã được kiểm tra.
- Hiểu vai trò của Amazon ElastiCache trong việc vận hành Redis và CloudWatch trong việc theo dõi log, metrics và tình trạng dịch vụ.

<!--
TODO: Add chat interface screenshots, Socket.IO event tests, DynamoDB table design, Redis adapter configuration, CloudWatch logs, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-4/
-->