---
title: "Nhật ký tuần 3"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Tuần 3 - Phát triển React frontend, tích hợp REST API và tìm hiểu triển khai tĩnh trên AWS

### Mục tiêu tuần 3:

- Xây dựng giao diện chính cho Candidate và HR/Company.
- Tích hợp React frontend với các REST API của backend.
- Hoàn thiện authentication context, role-based routing và protected pages.
- Chuẩn bị production build và xử lý cấu hình môi trường, CORS.
- Tìm hiểu cách triển khai static website bằng Amazon S3 và phân phối nội dung qua Amazon CloudFront.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Khởi tạo React với Vite, cấu hình Tailwind CSS, xây dựng cấu trúc thư mục frontend và các thành phần giao diện dùng chung. | 22/06/2026 | 22/06/2026 | [React Documentation](https://react.dev/learn); [Vite Getting Started](https://vite.dev/guide/); [Tailwind CSS with Vite](https://tailwindcss.com/docs/installation/using-vite) |
| 2 | Xây dựng trang đăng ký, đăng nhập, authentication context, lưu trạng thái người dùng và điều hướng theo vai trò Candidate hoặc HR/Company. | 23/06/2026 | 23/06/2026 | [React Documentation](https://react.dev/learn); [React Router Documentation](https://reactrouter.com/) |
| 3 | Phát triển Candidate dashboard, job board, profile, application detail và document management; tích hợp các API liên quan đến jobs, applications và documents. | 24/06/2026 | 24/06/2026 | [React Documentation](https://react.dev/learn); [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) |
| 4 | Phát triển HR dashboard, company profile, job editor và applicant review; bổ sung loading state, error handling và kiểm tra quyền truy cập cho các trang riêng tư. | 25/06/2026 | 25/06/2026 | [React Documentation](https://react.dev/learn); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 5 | Chuẩn bị production build, cấu hình biến môi trường và kiểm tra CORS; tìm hiểu cách lưu trữ static website trên Amazon S3 và phân phối nội dung thông qua Amazon CloudFront. | 26/06/2026 | 26/06/2026 | [Vite - Building for Production](https://vite.dev/guide/build); [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS); [Hosting a Static Website with Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html); [Amazon CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) |

### Kết quả đạt được trong tuần:

- Hoàn thiện các giao diện chính cho Candidate và HR/Company.
- Frontend có thể gọi các API authentication, jobs, applications, documents và dashboard.
- Người dùng được điều hướng theo đúng vai trò sau khi đăng nhập.
- Các trang riêng tư được bảo vệ bằng authentication state và role checks.
- Bổ sung xử lý loading, lỗi API và trường hợp phiên đăng nhập không hợp lệ.
- Production build được tạo và kiểm tra thành công.
- Hiểu cách Amazon S3 có thể lưu trữ static frontend và CloudFront hỗ trợ phân phối nội dung với tốc độ tốt hơn.

<!--
TODO: Add frontend screenshots, commits, API integration tests, production build output, S3 static hosting configuration, or CloudFront evidence for this week.
Expected image directory:
static/images/worklog/week-3/
-->