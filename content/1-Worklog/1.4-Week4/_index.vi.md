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

### Nội dung kỹ thuật đã triển khai:

Trong Tuần 4, tôi phát triển dịch vụ chat thời gian thực bằng Node.js, Express và Socket.IO. Dịch vụ cho phép Candidate và HR/Company tham gia nhóm chat, gửi tin nhắn, nhận tin nhắn mới ngay lập tức và kết nối lại sau khi mất mạng tạm thời.

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
DynamoDB và Redis
</pre>

### Luồng giao tiếp Socket.IO:

Sau khi người dùng đăng nhập, frontend mở kết nối Socket.IO bằng trạng thái xác thực hiện tại. Chat service kiểm tra kết nối trước khi cho phép người dùng tham gia room.

Khi người dùng gửi tin nhắn, server kiểm tra người gửi, lưu tin nhắn và phát sự kiện đến các thành viên trong nhóm chat.

<pre>
Người dùng gửi tin nhắn
        |
        v
Socket.IO event
        |
        v
Kiểm tra user và chat group
        |
        v
Lưu tin nhắn vào DynamoDB
        |
        v
Phát tin nhắn đến thành viên
</pre>

Frontend cũng lắng nghe sự kiện disconnect và reconnect để người dùng có thể tiếp tục cuộc trò chuyện sau khi kết nối mạng bị gián đoạn.

### Thiết kế dữ liệu DynamoDB:

Amazon DynamoDB được sử dụng để thiết kế mô hình lưu trữ cho người dùng chat, nhóm chat và tin nhắn.

Các bảng chính được tách theo trách nhiệm:

<pre>
ChatUsers
    |
    +-- Thông tin người dùng
    +-- Thông tin hiển thị

ChatGroups
    |
    +-- Thông tin nhóm
    +-- Thành viên nhóm

ChatMessages
    |
    +-- Group ID
    +-- Thời gian gửi
    +-- Người gửi
    +-- Nội dung
</pre>

Partition key và sort key được lựa chọn theo các truy vấn chính, đặc biệt là lấy danh sách nhóm chat của người dùng và tải lịch sử tin nhắn theo thứ tự thời gian.

### Redis Pub/Sub và nhiều service instance:

Một Socket.IO instance chỉ có thể phát sự kiện đến các client đang kết nối với chính instance đó. Khi hệ thống chạy nhiều instance, người dùng kết nối ở các instance khác nhau vẫn phải nhận được cùng một tin nhắn.

Socket.IO Redis adapter được tích hợp để đồng bộ sự kiện giữa các chat-service instance.

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
Người dùng nhận sự kiện
</pre>

Amazon ElastiCache for Redis được nghiên cứu như giải pháp Redis được AWS quản lý, hỗ trợ vận hành, giám sát và khả năng sẵn sàng tốt hơn.

### Tích hợp giao diện chat:

Giao diện React được kết nối với Socket.IO client và authentication state hiện có.

Người dùng có thể chọn nhóm chat, tải lịch sử tin nhắn, gửi tin nhắn mới và nhận cập nhật thời gian thực mà không cần tải lại trang.

Giao diện cũng có xử lý trạng thái kết nối để người dùng nhận biết khi realtime connection bị mất hoặc được khôi phục.

### Giám sát và Health Check:

Các endpoint health và metrics cơ bản được bổ sung để hỗ trợ việc theo dõi dịch vụ.

Chat service có thể cung cấp trạng thái hoạt động, số kết nối và một số thông tin runtime cơ bản. Amazon CloudWatch được nghiên cứu để thu thập log và giám sát hành vi của dịch vụ sau khi triển khai.

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

### Vấn đề và cách giải quyết:

| Vấn đề | Cách giải quyết | Trạng thái |
|---|---|---|
| Người dùng cần nhận tin nhắn ngay lập tức. | Triển khai Socket.IO event cho realtime communication. | Hoàn thành |
| Người dùng chưa xác thực có thể mở socket connection. | Bổ sung authentication middleware trước khi tham gia room. | Hoàn thành |
| Tin nhắn cần được lưu lâu dài. | Thiết kế bảng ChatMessages trong DynamoDB. | Hoàn thành |
| Nhiều chat instance không tự chia sẻ sự kiện. | Tích hợp Socket.IO Redis adapter. | Hoàn thành |
| Mất mạng tạm thời làm gián đoạn chat. | Bổ sung disconnect và reconnect handling. | Hoàn thành |
| Khó theo dõi hoạt động của realtime service. | Bổ sung health, metrics endpoint và nghiên cứu CloudWatch. | Hoàn thành |

### Kiến thức kỹ thuật đã học:

Tuần này giúp tôi hiểu sự khác biệt giữa REST API và giao tiếp realtime dựa trên event.

Tôi cũng hiểu rằng DynamoDB nên được thiết kế dựa trên access pattern, còn Redis Pub/Sub hỗ trợ đồng bộ sự kiện giữa nhiều service instance.

Một hệ thống realtime không chỉ cần gửi tin nhắn mà còn phải xử lý authentication, reconnect, lưu trữ và monitoring.

### Kết quả tuần:

Đến cuối Tuần 4, Candidate và HR/Company có thể trao đổi tin nhắn theo thời gian thực trên giao diện React.

Chat service hỗ trợ authenticated connection, room-based messaging, thiết kế lưu trữ DynamoDB, đồng bộ sự kiện qua Redis, reconnect và các endpoint giám sát cơ bản.

### Bài học rút ra:

Ứng dụng realtime cần sự phối hợp giữa frontend, WebSocket service, persistent storage và shared messaging infrastructure.

Socket.IO quản lý kết nối client, DynamoDB lưu dữ liệu chat, Redis đồng bộ nhiều service instance, còn CloudWatch hỗ trợ theo dõi vận hành.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung vào container hóa các service, chuẩn bị tài nguyên Kubernetes, cải thiện health check, configuration management và khả năng triển khai lặp lại.

<!--
TODO: Add chat interface screenshots, Socket.IO event tests, DynamoDB table design, Redis adapter configuration, CloudWatch logs, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-4/
-->