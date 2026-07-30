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

### Kết quả đạt được trong tuần:

- Các service chính đã được container hóa và có cấu hình runtime thống nhất.
- Hệ thống có thể được khởi động bằng Docker Compose trong môi trường local.
- Backend kết nối với PostgreSQL, trong khi chat service sử dụng Redis và DynamoDB Local.
- Các container giao tiếp với nhau thông qua internal network và service name.
- Database migration và demo data có thể được khởi tạo theo quy trình thống nhất.
- Health check và smoke test giúp xác nhận các service hoạt động sau khi khởi động.
- Local runbook và troubleshooting notes được chuẩn bị để hỗ trợ quá trình chạy lại hệ thống.

<!--
TODO: Add Docker container screenshots, Compose logs, health-check results, smoke-test evidence, or startup documentation for this week.
Expected image directory:
static/images/worklog/week-6/
-->