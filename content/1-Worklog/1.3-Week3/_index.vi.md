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

### Nội dung kỹ thuật đã triển khai:

Trong Tuần 3, tôi phát triển giao diện React dành cho Ứng viên và HR/Doanh nghiệp, đồng thời kết nối frontend với các REST API của backend.

Frontend được khởi tạo bằng React và Vite. Tailwind CSS được sử dụng để xây dựng layout, form, bảng, card và navigation nhất quán. Source code được chia thành các nhóm như pages, reusable components, authentication context, routing và API services.

<pre>
Trình duyệt
    |
    v
React Frontend
    |
    +-- Authentication Context
    +-- React Router
    +-- Trang Candidate
    +-- Trang HR/Company
    +-- API Service Layer
    |
    v
Backend REST APIs
</pre>

### Authentication và phân quyền theo vai trò:

Trang đăng ký và đăng nhập được kết nối với authentication API. Sau khi đăng nhập thành công, frontend lưu trạng thái người dùng và điều hướng theo vai trò.

Protected route kiểm tra cả trạng thái đăng nhập và role. Candidate được chuyển đến các chức năng dành cho Ứng viên, trong khi HR/Company truy cập các trang quản lý doanh nghiệp và tuyển dụng.

<pre>
Login Form
    |
    v
Authentication API
    |
    +-- Session hợp lệ
    |       |
    |       v
    |   Đọc user role
    |       |
    |       v
    |   Chuyển đến dashboard
    |
    +-- Session không hợp lệ
            |
            v
        Hiển thị lỗi
</pre>

Việc bảo vệ route ở frontend hỗ trợ trải nghiệm người dùng, nhưng backend vẫn chịu trách nhiệm kiểm tra quyền truy cập thực tế.

### Chức năng dành cho Candidate:

Candidate có thể quản lý hồ sơ, xem danh sách công việc, xem chi tiết công việc, nộp đơn ứng tuyển, tải tài liệu và theo dõi trạng thái ứng tuyển.

Các trang Candidate giao tiếp với profile, job, application và document API thông qua các service dùng chung.

<pre>
Candidate Dashboard
    |
    +-- Quản lý hồ sơ
    +-- Xem công việc
    +-- Xem chi tiết công việc
    +-- Nộp đơn ứng tuyển
    +-- Tải tài liệu
    +-- Theo dõi trạng thái
</pre>

### Chức năng dành cho HR và Company:

HR/Company có thể quản lý thông tin doanh nghiệp, tạo và cập nhật tin tuyển dụng, xem danh sách ứng viên, kiểm tra hồ sơ, truy cập tài liệu được cấp quyền và cập nhật trạng thái ứng tuyển.

<pre>
HR Dashboard
    |
    +-- Quản lý hồ sơ công ty
    +-- Tạo tin tuyển dụng
    +-- Cập nhật tin tuyển dụng
    +-- Xem danh sách ứng viên
    +-- Xem tài liệu
    +-- Cập nhật trạng thái
</pre>

### Tích hợp REST API:

Các request được tổ chức trong một API service layer riêng thay vì viết trực tiếp trong từng page component.

Cấu trúc này giúp quản lý endpoint, request header, payload, authentication và error handling một cách nhất quán.

<pre>
React Page
    |
    v
API Service Function
    |
    +-- Tạo request
    +-- Thêm authentication
    +-- Gửi request
    +-- Xử lý response
    |
    v
Backend REST API
    |
    v
Cập nhật React State
</pre>

Loading state và error message được bổ sung để tránh hiển thị dữ liệu chưa hoàn chỉnh. Khi session không còn hợp lệ, frontend xóa trạng thái đăng nhập và chuyển người dùng về trang login.

### Production Build và Environment Configuration:

Backend API URL được cấu hình bằng Vite environment variables để môi trường development và production có thể sử dụng endpoint khác nhau.

Production build chuyển source code React thành các file HTML, JavaScript, CSS và asset đã được tối ưu.

<pre>
React Source Code
        |
        v
Vite Production Build
        |
        +-- Bundle JavaScript
        +-- Xử lý CSS
        +-- Xử lý assets
        +-- Áp dụng environment values
        |
        v
Static Build Files
</pre>

CORS cũng được kiểm tra vì frontend và backend có thể chạy trên các domain khác nhau. Trong production, backend chỉ nên cho phép các frontend origin đáng tin cậy.

### Amazon S3 và CloudFront:

Amazon S3 được nghiên cứu để lưu các static file của frontend. Vì production build chỉ gồm các file tĩnh, frontend có thể được host mà không cần một application server riêng.

Amazon CloudFront có thể cache và phân phối nội dung qua các edge location, giúp giảm độ trễ và hỗ trợ HTTPS.

<pre>
Trình duyệt người dùng
          |
          v
Amazon CloudFront
          |
          +-- Đã có file trong cache
          |            |
          |            v
          |      Trả file từ cache
          |
          +-- Chưa có file trong cache
                       |
                       v
                Amazon S3 Bucket
                       |
                       v
                 Trả static file
</pre>

Đối với React Router, S3 và CloudFront cần trả về `index.html` cho các route như `/jobs` hoặc `/dashboard`. Nếu không, việc refresh trực tiếp một route có thể gây lỗi không tìm thấy trang.

### Vấn đề và cách giải quyết:

| Vấn đề | Cách giải quyết | Trạng thái |
|---|---|---|
| Candidate và HR cần giao diện khác nhau. | Bổ sung role-based navigation và protected route. | Hoàn thành |
| Logic gọi API bị lặp lại giữa các trang. | Tạo các API service module dùng chung. | Hoàn thành |
| Trang được render trước khi API trả dữ liệu. | Bổ sung loading state và conditional rendering. | Hoàn thành |
| Lỗi API chưa được hiển thị rõ ràng. | Bổ sung error message và invalid-session handling. | Hoàn thành |
| Development và production dùng API URL khác nhau. | Chuyển backend URL sang Vite environment configuration. | Hoàn thành |
| Request khác origin có thể bị trình duyệt chặn. | Kiểm tra và cấu hình CORS cho frontend origin. | Hoàn thành |
| React route có thể lỗi khi refresh trang. | Chuẩn bị cơ chế fallback về `index.html`. | Hoàn thành |

### Kiến thức kỹ thuật đã học:

Tuần này giúp tôi hiểu cách phân tách frontend thành component, routing, authentication, state management và API service.

Tôi cũng hiểu rằng kiểm tra role ở frontend chỉ hỗ trợ điều hướng và không thể thay thế authorization ở backend.

Production build cho thấy React có thể được chuyển thành static file để lưu trên Amazon S3 và phân phối qua Amazon CloudFront.

### Kết quả tuần:

Đến cuối Tuần 3, các giao diện chính dành cho Candidate và HR/Company đã được hoàn thành và tích hợp với backend API.

Authentication context, protected routing, loading state, error handling, environment configuration và production build cũng đã được chuẩn bị.

Kiến trúc triển khai dự kiến sử dụng Amazon S3 để lưu frontend và Amazon CloudFront để phân phối nội dung an toàn, nhanh chóng hơn.

### Bài học rút ra:

Một frontend hoàn chỉnh không chỉ bao gồm giao diện mà còn phải xử lý authentication, routing, API communication, loading, lỗi, environment configuration và deployment.

Amazon S3 và CloudFront có vai trò bổ trợ cho nhau: S3 lưu static files, còn CloudFront cache và phân phối các file đó đến người dùng.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung vào containerization, quy trình build có thể lặp lại, automated testing, security checks và chuẩn bị triển khai trong môi trường AWS.

<!--
TODO: Add frontend screenshots, commits, API integration tests, production build output, S3 static hosting configuration, or CloudFront evidence for this week.
Expected image directory:
static/images/worklog/week-3/
-->