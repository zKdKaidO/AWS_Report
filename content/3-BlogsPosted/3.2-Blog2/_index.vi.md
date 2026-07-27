---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# XÂY DỰNG HỆ THỐNG CHAT REALTIME CÓ KHẢ NĂNG MỞ RỘNG VỚI SOCKET.IO, REDIS VÀ DYNAMODB

Chat realtime hiện nay xuất hiện trong rất nhiều hệ thống như thương mại điện tử, tuyển dụng, chăm sóc khách hàng, mạng xã hội và các nền tảng cộng tác.

Ở mức cơ bản, chúng ta chỉ cần một server để nhận và gửi tin nhắn. Tuy nhiên, khi số lượng người dùng tăng lên, hệ thống bắt đầu gặp những câu hỏi khó hơn:

- Làm thế nào để tin nhắn xuất hiện ngay lập tức?
- Điều gì xảy ra khi người gửi và người nhận kết nối vào hai server khác nhau?
- Tin nhắn được lưu ở đâu để không bị mất khi server khởi động lại?
- Làm thế nào để tăng số lượng server mà không làm gián đoạn người dùng?

Một kiến trúc phổ biến để giải quyết những vấn đề này là kết hợp Socket.IO, Redis và Amazon DynamoDB.

## 1. Vì sao REST API chưa đủ cho chat realtime?

REST API hoạt động theo mô hình request-response:

```text
Client gửi request
↓
Server xử lý
↓
Server trả response
```

Nếu dùng REST API cho chat, frontend thường phải liên tục gọi API để kiểm tra tin nhắn mới:

```text
GET /messages
Chờ vài giây
GET /messages
Chờ vài giây
GET /messages
```

Cách này gọi là polling.

Polling tương đối đơn giản nhưng có một số hạn chế:

- Tin nhắn có độ trễ phụ thuộc vào chu kỳ gọi API.
- Client gửi request ngay cả khi không có tin nhắn mới.
- Backend phải xử lý nhiều request không cần thiết.
- Khi số lượng người dùng tăng, tải hệ thống cũng tăng nhanh.

Đối với hệ thống cần phản hồi gần như tức thời, WebSocket hoặc một thư viện hỗ trợ realtime như Socket.IO thường phù hợp hơn.

## 2. Socket.IO được dùng để giao tiếp realtime

Socket.IO cho phép duy trì kết nối hai chiều giữa client và server.

Sau khi kết nối được thiết lập, client và server có thể chủ động gửi sự kiện cho nhau mà không cần tạo một request HTTP mới cho mỗi lần trao đổi.

Luồng gửi tin nhắn có thể hiểu đơn giản như sau:

```text
Người dùng A gửi tin nhắn
↓
Socket.IO server nhận sự kiện
↓
Server xử lý tin nhắn
↓
Server gửi sự kiện mới
↓
Người dùng B nhận tin nhắn
```

Một số sự kiện thường gặp:

- `conversation:join`
- `message:send`
- `message:new`
- `user:typing`
- `message:read`

Socket.IO còn hỗ trợ:

- Tự động kết nối lại khi mạng bị gián đoạn.
- Chia người dùng vào các room.
- Gửi sự kiện đến một người hoặc một nhóm người.
- Fallback khi WebSocket không khả dụng.

Tuy nhiên, Socket.IO chỉ giải quyết tốt bài toán realtime khi hệ thống có một server. Khi ứng dụng chạy nhiều server, chúng ta cần thêm một cơ chế đồng bộ.

## 3. Vấn đề xuất hiện khi chat service chạy nhiều instance

Giả sử hệ thống chỉ có một chat server:

```text
Người dùng A ─┐
              ├── Chat server
Người dùng B ─┘
```

Cả hai người dùng đều kết nối vào cùng một server. Khi người dùng A gửi tin nhắn, server có thể gửi trực tiếp đến kết nối của người dùng B.

Khi số lượng người dùng tăng, hệ thống có thể phải chạy nhiều server:

```text
Chat server 1
Chat server 2
Chat server 3
```

Load balancer có thể phân phối người dùng như sau:

```text
Người dùng A → Chat server 1
Người dùng B → Chat server 2
```

Lúc này, Chat server 1 không trực tiếp quản lý kết nối của người dùng B.

Nếu không có cơ chế liên lạc giữa các server, tin nhắn của người dùng A có thể được lưu thành công nhưng người dùng B không nhận được thông báo realtime.

Đây là vấn đề mà Redis có thể giải quyết.

## 4. Redis giúp đồng bộ sự kiện giữa các chat server

Redis là một hệ thống lưu trữ dữ liệu trong bộ nhớ có tốc độ cao. Trong kiến trúc chat realtime, Redis thường được sử dụng cho cơ chế publish/subscribe.

Luồng hoạt động:

```text
Người dùng A
↓
Chat server 1
↓ Publish
Redis
↓ Subscribe
Chat server 2
↓
Người dùng B
```

Khi Chat server 1 nhận được tin nhắn:

- Server xử lý và lưu tin nhắn.
- Server phát sự kiện lên Redis.
- Các chat server khác nhận được sự kiện.
- Server đang giữ kết nối của người nhận gửi tin nhắn xuống client.

Có thể hiểu Redis giống như một “kênh phát thanh” dùng chung giữa các chat server.

Redis phù hợp với nhiệm vụ này vì:

- Tốc độ xử lý cao.
- Hỗ trợ publish/subscribe.
- Phù hợp cho dữ liệu và sự kiện ngắn hạn.
- Giúp các server hoạt động độc lập nhưng vẫn trao đổi được với nhau.

Redis trong trường hợp này không nhất thiết là nơi lưu lịch sử chat lâu dài. Nhiệm vụ đó nên được giao cho một database bền vững hơn.

## 5. DynamoDB được dùng để lưu trữ dữ liệu chat

Amazon DynamoDB là dịch vụ NoSQL database được quản lý bởi AWS.

Dữ liệu chat thường có một số đặc điểm:

- Số lượng tin nhắn tăng liên tục.
- Tốc độ ghi có thể cao.
- Tin nhắn thường được truy vấn theo cuộc hội thoại.
- Ít cần các phép join phức tạp.
- Nhu cầu sử dụng có thể tăng đột biến.

DynamoDB phù hợp với mô hình này vì có khả năng mở rộng tốt và không yêu cầu người dùng tự quản lý server database.

Một thiết kế đơn giản cho bảng tin nhắn có thể sử dụng:

- Partition key: `conversation_id`
- Sort key: `timestamp` hoặc `message_id`

Trong đó:

- `conversation_id` dùng để nhóm các tin nhắn của cùng một cuộc hội thoại.
- `timestamp` dùng để sắp xếp tin nhắn theo thời gian.

Khi cần lấy lịch sử chat, hệ thống truy vấn các bản ghi có cùng `conversation_id`.

Vai trò của Redis và DynamoDB có thể phân biệt như sau:

```text
Redis
→ Truyền sự kiện realtime giữa các server

DynamoDB
→ Lưu trữ lịch sử tin nhắn
```

## 6. Luồng gửi tin nhắn hoàn chỉnh

Một tin nhắn có thể đi qua hệ thống theo các bước sau:

1. Người dùng nhập nội dung tin nhắn.
2. Client gửi sự kiện qua Socket.IO.
3. Chat server xác thực người dùng.
4. Server kiểm tra quyền tham gia cuộc hội thoại.
5. Tin nhắn được lưu vào DynamoDB.
6. Server publish sự kiện qua Redis.
7. Chat server đang giữ kết nối của người nhận nhận sự kiện.
8. Server gửi tin nhắn mới đến client của người nhận.

Sơ đồ tổng quát:

```text
Client A
│
│ Socket.IO
▼
Chat server 1
│
├──── Lưu dữ liệu ─────► DynamoDB
│
└──── Publish event ───► Redis
                         │
                         ▼
                  Chat server 2
                         │
                         ▼
                      Client B
```

Kiến trúc này giúp hệ thống đạt được hai mục tiêu:

- Phản hồi realtime nhờ Socket.IO và Redis.
- Không mất lịch sử tin nhắn nhờ DynamoDB.

## 7. Kubernetes hỗ trợ scale chat service như thế nào?

Khi ứng dụng được đóng gói bằng container, Kubernetes có thể chạy nhiều bản sao của chat service.

```text
Chat service
├── Pod 1
├── Pod 2
├── Pod 3
└── Pod N
```

Kubernetes Service hoặc load balancer phân phối kết nối đến các pod.

Kubernetes còn hỗ trợ:

- Tự động khởi động lại container khi xảy ra lỗi.
- Rolling update khi triển khai phiên bản mới.
- Tăng hoặc giảm số lượng pod.
- Health check thông qua liveness và readiness probe.
- Phân phối traffic giữa nhiều pod.
- Horizontal Pod Autoscaler dựa trên CPU, memory hoặc custom metrics.

Tuy nhiên, Kubernetes chỉ giúp chạy và quản lý nhiều instance. Việc đồng bộ realtime giữa các instance vẫn cần Redis hoặc một message broker phù hợp.

## 8. Sticky session và Redis giải quyết hai vấn đề khác nhau

Khi Socket.IO sử dụng HTTP long polling, một client có thể tạo nhiều request liên tiếp trong cùng một phiên.

Nếu các request bị chuyển đến nhiều server khác nhau, phiên kết nối có thể gặp lỗi.

Sticky session giúp các request của cùng một client tiếp tục được chuyển về cùng một server:

```text
Client A → Chat server 1
Client A → Chat server 1
Client A → Chat server 1
```

Tuy nhiên, sticky session không thay thế Redis.

Có thể phân biệt:

- Sticky session giữ kết nối của một client ổn định trên một server.
- Redis truyền sự kiện giữa các server khác nhau.

Ngay cả khi đã bật sticky session, người gửi và người nhận vẫn có thể nằm trên hai server khác nhau.

## 9. Triển khai trên AWS

Khi triển khai trên AWS, kiến trúc có thể sử dụng:

```text
Application Load Balancer
↓
Amazon EKS hoặc container service
↓
Nhiều chat service instance
├── Amazon DynamoDB
└── Amazon ElastiCache/Valkey
```

Vai trò của các dịch vụ:

- Amazon EKS: chạy và quản lý các container chat service.
- Application Load Balancer: nhận và phân phối kết nối từ người dùng.
- Amazon DynamoDB: lưu dữ liệu chat.
- Amazon ElastiCache hoặc Valkey: cung cấp Redis-compatible service.
- Amazon CloudWatch: lưu log, metrics và cảnh báo.

Với project nhỏ hoặc môi trường thử nghiệm, có thể chạy Redis và DynamoDB Local bằng Docker trước khi sử dụng dịch vụ AWS thật.

## 10. Những lỗi thường gặp khi xây dựng chat realtime

### Chỉ lưu tin nhắn trong bộ nhớ của server

Nếu server restart, toàn bộ dữ liệu có thể bị mất.

```text
Server restart → Message history mất
```

Tin nhắn cần được lưu vào một database bền vững.

### Chạy nhiều server nhưng không có Redis adapter

Mỗi server chỉ biết các client đang kết nối trực tiếp với nó. Người dùng ở server khác có thể không nhận được sự kiện.

### Sử dụng Redis làm nơi lưu trữ duy nhất

Redis phù hợp cho cache và truyền sự kiện, nhưng không phải lúc nào cũng phù hợp để trở thành nguồn lưu trữ lịch sử chat duy nhất.

### Không xác thực Socket.IO connection

WebSocket hoặc Socket.IO connection vẫn cần được xác thực bằng JWT, session hoặc một cơ chế phù hợp.

### Không kiểm tra quyền truy cập conversation

Một người dùng đã đăng nhập không có nghĩa là họ được phép tham gia mọi phòng chat.

### Không xử lý tin nhắn gửi trùng

Khi mạng không ổn định, client có thể gửi lại cùng một message. Có thể sử dụng client-generated message ID hoặc idempotency key để hạn chế bản ghi trùng.

### Không giới hạn kích thước và tần suất gửi tin nhắn

Hệ thống nên có:

- Giới hạn độ dài message.
- Rate limiting.
- Chống spam.
- Kiểm tra loại file nếu hỗ trợ attachment.

## 11. Một số cách tối ưu chi phí

Hệ thống chat realtime trên AWS có thể phát sinh chi phí từ compute, load balancer, database, Redis và log.

Một số cách tối ưu:

- Sử dụng DynamoDB On-Demand khi traffic còn thấp hoặc khó dự đoán.
- Chọn ElastiCache node phù hợp thay vì sử dụng cấu hình quá lớn.
- Dùng Auto Scaling để tránh chạy quá nhiều instance khi traffic thấp.
- Đặt thời gian lưu CloudWatch Logs thay vì giữ log vô thời hạn.
- Sử dụng DynamoDB TTL cho dữ liệu tạm thời nếu nghiệp vụ cho phép.
- Kiểm tra và xóa Load Balancer, ElastiCache cluster hoặc môi trường thử nghiệm sau khi hoàn thành lab.
- Chạy hệ thống bằng Docker và DynamoDB Local trong giai đoạn phát triển.

Việc sử dụng managed services giúp giảm công sức vận hành, nhưng vẫn cần theo dõi tài nguyên để tránh phát sinh chi phí ngoài dự kiến.

## 12. Checklist tham khảo

- Sử dụng Socket.IO hoặc WebSocket cho giao tiếp realtime.
- Dùng Redis adapter khi chạy nhiều chat server.
- Lưu lịch sử tin nhắn trong database bền vững.
- Thiết kế partition key DynamoDB phù hợp.
- Không lưu state quan trọng chỉ trong memory của server.
- Cấu hình sticky session nếu sử dụng long polling.
- Xác thực kết nối realtime.
- Kiểm tra quyền tham gia cuộc hội thoại.
- Thêm rate limiting và giới hạn kích thước tin nhắn.
- Cấu hình liveness và readiness probe.
- Theo dõi số connection, message rate, latency và error rate.
- Thiết lập Auto Scaling và resource limit phù hợp.
- Kiểm tra chi phí các tài nguyên AWS sau khi thử nghiệm.

Bài học quan trọng nhất mình rút ra là: chat realtime không chỉ là thiết lập một kết nối WebSocket.

Khi hệ thống bắt đầu mở rộng, chúng ta cần tách rõ ba nhiệm vụ:

```text
Socket.IO
→ Giao tiếp realtime giữa client và server

Redis
→ Đồng bộ sự kiện giữa nhiều server

DynamoDB
→ Lưu trữ dữ liệu chat lâu dài
```

Việc lựa chọn đúng công cụ cho từng nhiệm vụ giúp hệ thống vừa phản hồi nhanh, vừa có khả năng mở rộng khi số lượng người dùng tăng.

`#AWS` `#SocketIO` `#Redis` `#DynamoDB` `#AmazonEKS` `#RealtimeChat` `#NodeJS` `#Kubernetes` `#CloudComputing` `#AWSStudyGroup`
