---
title: "Nhật ký tuần 6"
date: 2026-07-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Tuần 6 - Container hóa hệ thống và tích hợp với Docker Compose

### Mục tiêu tuần 6:

- Chuẩn hóa môi trường chạy cho backend, frontend, chat service và AI service.
- Tích hợp các thành phần của hệ thống trong cùng một môi trường Docker Compose.
- Hoàn thiện quy trình khởi động, kiểm tra tình trạng dịch vụ và chuẩn bị dữ liệu demo.
- Xây dựng hướng dẫn chạy hệ thống local có thể sử dụng lại.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Viết và điều chỉnh Dockerfile cho backend, frontend, chat service và AI service; chuẩn hóa biến môi trường, thư mục làm việc, dependency installation và startup command cho từng container. | 13/07/2026 | 13/07/2026 | [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/); [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/) |
| 2 | Cấu hình PostgreSQL, Redis và DynamoDB Local trong Docker Compose; tích hợp backend, worker, frontend và chat service thông qua internal network, volume và service dependencies. | 15/07/2026 | 16/07/2026 | [Docker Compose Application Model](https://docs.docker.com/compose/intro/compose-application-model/); [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/); [Control Startup Order](https://docs.docker.com/compose/how-tos/startup-order/) |
| 3 | Tách cấu hình AI service thành Compose file riêng, chạy database migration và seed data, sau đó thực hiện health check, smoke test và ghi lại các lỗi thường gặp trong quá trình khởi động hệ thống. | 17/07/2026 | 17/07/2026 | [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/); [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/); [Control Startup Order](https://docs.docker.com/compose/how-tos/startup-order/) |

### Nội dung kỹ thuật đã triển khai:

Trong Tuần 6, tôi container hóa backend, frontend, chat service và AI service để mỗi thành phần có thể chạy trong một môi trường nhất quán và độc lập.

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

Mỗi service có Dockerfile riêng, trong đó xác định working directory, dependency installation, environment configuration, exposed port và startup command.

### Tích hợp Docker Compose:

Docker Compose được sử dụng để kết nối các service chính trong cùng một môi trường local.

Backend kết nối với PostgreSQL, trong khi chat service sử dụng Redis và DynamoDB Local. Các container giao tiếp thông qua service name thay vì IP cố định.

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

Volume được bổ sung cho những dữ liệu cần lưu lâu dài. Service dependency và health check được sử dụng để kiểm soát thứ tự khởi động.

### Quy trình khởi động và Health Check:

Một quy trình khởi động có thể lặp lại được chuẩn bị để giảm các bước cấu hình thủ công.

<pre>
Build container images
        |
        v
Khởi động infrastructure services
        |
        v
Chạy database migrations
        |
        v
Tạo demo data
        |
        v
Khởi động application services
        |
        v
Chạy health checks và smoke tests
</pre>

Health endpoint được sử dụng để kiểm tra service đã chạy và đã kết nối được với các dependency cần thiết.

### Tách riêng AI Service:

AI service được đặt trong một Compose file riêng vì có thể yêu cầu dependency, tài nguyên hoặc startup workflow khác với hệ thống chính.

Cách tách này cho phép chạy core system mà không cần luôn khởi động AI workload.

### Vấn đề và cách giải quyết:

| Vấn đề | Cách giải quyết | Trạng thái |
|---|---|---|
| Service hoạt động khác nhau trên từng máy. | Chuẩn hóa runtime bằng Dockerfile. | Hoàn thành |
| Local setup cần nhiều lệnh thủ công. | Tích hợp các service bằng Docker Compose. | Hoàn thành |
| Container không thể dùng localhost để gọi nhau. | Sử dụng service name và internal network. | Hoàn thành |
| Backend có thể khởi động trước khi PostgreSQL sẵn sàng. | Bổ sung service dependency và health check. | Hoàn thành |
| Dữ liệu local bị mất khi tạo lại container. | Bổ sung volume cho dữ liệu cần lưu. | Hoàn thành |
| Database và demo data chưa được khởi tạo nhất quán. | Chuẩn hóa migration và seeding. | Hoàn thành |
| AI dependency làm Compose chính nặng hơn. | Tách AI service sang Compose file riêng. | Hoàn thành |

### Kiến thức kỹ thuật đã học:

Tuần này giúp tôi hiểu cách container tạo môi trường runtime nhất quán giữa nhiều máy.

Tôi cũng học được cách Docker Compose quản lý network, environment variables, volumes, startup dependency và health check.

Một hệ thống chạy được không chỉ cần container mà còn cần migration, demo data, smoke test và tài liệu troubleshooting rõ ràng.

### Kết quả tuần:

Đến cuối Tuần 6, các thành phần chính của hệ thống có thể chạy local thông qua Docker Compose.

Backend đã kết nối với PostgreSQL, còn chat service kết nối với Redis và DynamoDB Local. Quy trình startup cũng bao gồm migration, demo data, health check và smoke test.

### Bài học rút ra:

Containerization giúp giảm khác biệt môi trường, nhưng hệ thống vẫn cần quản lý dependency và cấu hình rõ ràng.

Docker Compose hỗ trợ tích hợp local bằng cách đặt các service trong cùng một network và cho phép giao tiếp qua service name.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung vào Kubernetes deployment, configuration, secrets, scaling và observability cho các containerized service.

<!--
TODO: Add Docker container screenshots, Compose logs, health-check results, smoke-test evidence, or startup documentation for this week.
Expected image directory:
static/images/worklog/week-6/
-->