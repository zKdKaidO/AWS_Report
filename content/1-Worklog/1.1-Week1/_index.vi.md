---
title: "Nhật ký tuần 1"
date: 2026-06-08
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Tuần 1 - Định hướng chương trình, phân tích hệ thống và làm quen với AWS

### Mục tiêu tuần 1:

- Hiểu yêu cầu thực tập, cấu trúc workshop mẫu và quy định báo cáo project FCAJ.
- Phân tích bài toán quản lý tuyển dụng và ứng tuyển thực tập.
- Xác định các nhóm người dùng, chức năng chính và kiến trúc tổng quan của hệ thống.
- Làm quen với AWS Account, AWS Budgets và nguyên tắc quản lý quyền truy cập bằng AWS Identity and Access Management.
- Khởi tạo nền tảng backend và authentication flow ban đầu.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Tìm hiểu yêu cầu chương trình thực tập, cấu trúc workshop mẫu và quy định trình bày báo cáo project FCAJ. | 08/06/2026 | 08/06/2026 | [FCAJ Project Requirements](https://cloudjourney.awsstudygroup.com/8-fcjworkforce/); [FCAJ Internship Report Sample](https://workshop-sample.awsfcaj.com) |
| 2 | Phân tích bài toán tuyển dụng, xác định hai nhóm người dùng chính gồm Candidate và HR/Company, đồng thời xây dựng sơ bộ các luồng đăng tuyển, tìm kiếm công việc, nộp hồ sơ và quản lý ứng viên. | 09/06/2026 | 10/06/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html); [FCAJ Project Requirements](https://cloudjourney.awsstudygroup.com/8-fcjworkforce/) |
| 3 | Tạo và kiểm tra AWS Account, tìm hiểu cách theo dõi chi phí bằng AWS Budgets và nguyên tắc cấp quyền tối thiểu trong AWS IAM. | 11/06/2026 | 11/06/2026 | [Creating an AWS Account](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html); [Managing Costs with AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html); [Introduction to AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) |
| 4 | Khởi tạo FastAPI backend, cấu hình database connection, tạo các model ban đầu, chuẩn bị Alembic migration và xây dựng registration, login, password hashing cùng JWT authentication. | 12/06/2026 | 12/06/2026 | [FastAPI OAuth2 with JWT](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/); [SQLAlchemy ORM Quick Start](https://docs.sqlalchemy.org/en/20/orm/quickstart.html); [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html) |

### Kiến thức và nội dung kỹ thuật đã tìm hiểu:

Trong tuần đầu tiên, tôi tập trung tìm hiểu dự án từ cả góc độ nghiệp vụ và kỹ thuật trước khi bắt đầu các hoạt động triển khai lên AWS.

Tôi xem xét yêu cầu của chương trình thực tập, cấu trúc workshop FCAJ và hướng dẫn viết báo cáo để hiểu rõ các kết quả cần đạt được. Đồng thời, tôi phân tích quy trình tuyển dụng và quản lý ứng tuyển thực tập, từ đó xác định hai nhóm người dùng chính là Ứng viên và HR/Doanh nghiệp.

Ứng viên cần tìm kiếm công việc, nộp hồ sơ, tải tài liệu, trao đổi với nhà tuyển dụng và nhận các đề xuất công việc phù hợp. HR/Doanh nghiệp cần tạo tin tuyển dụng, quản lý hồ sơ ứng tuyển, xem xét thông tin ứng viên, trao đổi với ứng viên và đánh giá mức độ phù hợp giữa ứng viên với công việc.

Về kỹ thuật, tôi nghiên cứu cấu trúc ban đầu của hệ thống và xác định các thành phần chính:

- Frontend React và Vite
- Backend FastAPI
- Dịch vụ chat Node.js và Socket.IO
- Dịch vụ xử lý AI
- Cơ sở dữ liệu quan hệ PostgreSQL
- Redis dùng cho caching và phân phối tin nhắn thời gian thực
- DynamoDB dùng để lưu dữ liệu liên quan đến chat
- Hệ thống lưu trữ tài liệu

Tôi cũng xem xét luồng xác thực của backend, bao gồm đăng ký tài khoản, mã hóa mật khẩu, kiểm tra đăng nhập, tạo JWT access token, bảo vệ API, xây dựng model bằng SQLAlchemy và quản lý database migration bằng Alembic.

### Kiến thức AWS đã học:

Trong Tuần 1, tôi tìm hiểu một số kiến thức AWS nền tảng cần thiết cho các giai đoạn triển khai tiếp theo.

Tôi học cách một AWS Account được tổ chức và lý do tài khoản root chỉ nên được sử dụng cho các tác vụ quản trị cấp tài khoản. Các hoạt động phát triển và triển khai thông thường nên sử dụng IAM user hoặc IAM role với quyền hạn phù hợp.

Tôi nghiên cứu AWS Identity and Access Management, bao gồm IAM user, group, role, policy, permission và nguyên tắc cấp quyền tối thiểu. Nguyên tắc này yêu cầu mỗi người dùng hoặc dịch vụ chỉ được cấp các quyền cần thiết để thực hiện trách nhiệm được giao.


Bên cạnh đó, tôi xem xét AWS Well-Architected Framework và sáu trụ cột chính:

- Vận hành hiệu quả
- Bảo mật
- Độ tin cậy
- Hiệu năng
- Tối ưu chi phí
- Tính bền vững

Các nguyên tắc này tạo ra cơ sở ban đầu để đánh giá kiến trúc hệ thống trong những giai đoạn triển khai sau.

### Kiến trúc cơ sở ban đầu:

Đến cuối Tuần 1, tôi đã xác định được luồng logic ban đầu của ứng dụng.

Ứng viên và HR truy cập frontend React/Vite thông qua trình duyệt web. Frontend giao tiếp với backend FastAPI để thực hiện xác thực, quản lý công việc, quản lý hồ sơ ứng tuyển, xử lý tài liệu và các chức năng liên quan đến AI.

Giao tiếp thời gian thực được xử lý bởi một dịch vụ Node.js và Socket.IO riêng biệt. PostgreSQL lưu dữ liệu quan hệ của ứng dụng, Redis hỗ trợ caching và phân phối tin nhắn thời gian thực, DynamoDB lưu các bản ghi liên quan đến chat, còn dịch vụ AI thực hiện xử lý CV, chuẩn hóa dữ liệu công việc, ghép nối ứng viên và reranking.

Kiến trúc logic ban đầu có thể được tóm tắt như sau:

<pre>
Ứng viên / HR
      |
      v
Frontend React / Vite
      |
      +-----------------------------+
      |                             |
      v                             v
Backend FastAPI            Dịch vụ Chat Socket.IO
      |                             |
      v                             v
PostgreSQL                 Redis và DynamoDB
      |
      v
Lưu trữ tài liệu và Dịch vụ AI
</pre>

Đây mới chỉ là kiến trúc cơ sở dành cho môi trường phát triển cục bộ. Kiến trúc này được xây dựng để hiểu rõ trách nhiệm và sự phụ thuộc của từng dịch vụ trước khi ánh xạ chúng sang hạ tầng AWS.

Quá trình phân tích cũng xác định hướng chuyển đổi lên cloud ban đầu:

- Host frontend dưới dạng nội dung web tĩnh.
- Container hóa backend, dịch vụ chat và dịch vụ AI.
- Chuyển dữ liệu quan hệ sang Amazon RDS for PostgreSQL.
- Sử dụng Amazon S3 để lưu trữ tài liệu.
- Sử dụng Amazon DynamoDB để lưu dữ liệu liên quan đến chat.
- Sử dụng Redis hoặc Amazon ElastiCache cho giao tiếp thời gian thực và caching.
- Chuẩn bị các dịch vụ cho quy trình build và triển khai tự động thông qua CI/CD.

### Vấn đề và cách giải quyết:

| Vấn đề | Nguyên nhân | Cách giải quyết | Trạng thái |
|---|---|---|---|
| Dự án chứa nhiều dịch vụ và module nghiệp vụ nên ban đầu khó hiểu toàn bộ kiến trúc. | Repository bao gồm frontend, backend, chat, AI, cơ sở dữ liệu và hệ thống lưu trữ. | Tôi chia hệ thống thành các ranh giới dịch vụ logic và ghi lại trách nhiệm của từng thành phần. | Hoàn thành |
| Tôi chưa có nhiều kinh nghiệm thực tế với AWS IAM và giám sát chi phí. | Các công việc trước đó chủ yếu tập trung vào phát triển ứng dụng cục bộ. | Tôi nghiên cứu AWS IAM, nguyên tắc cấp quyền tối thiểu, AWS Budgets và các thực hành bảo mật tài khoản cơ bản. | Hoàn thành |
| Luồng xác thực bao gồm nhiều thành phần chưa quen thuộc. | Xác thực yêu cầu mã hóa mật khẩu, JWT, API được bảo vệ, database model và migration. | Tôi xem xét luồng xác thực FastAPI theo từng bước, từ đăng ký đến truy cập API được bảo vệ. | Hoàn thành |
| Tuần 1 chưa có bằng chứng triển khai AWS. | Tuần đầu tập trung vào định hướng, phân tích hệ thống và khởi tạo backend. | Tôi ghi lại kiến trúc cơ sở và chuẩn bị hướng triển khai cloud thay vì khai báo những hoạt động triển khai chưa thực hiện. | Đã lên kế hoạch |
| Screenshot và terminal log chưa được lưu trong thư mục bằng chứng của worklog. | Quy trình thu thập bằng chứng chưa được chuẩn hóa trong tuần đầu. | Tôi chuẩn bị checklist bằng chứng gồm cấu hình AWS, source code, test, Git commit và quá trình chạy ứng dụng cục bộ. | Đang chờ |

### Kết quả kiểm thử, build và triển khai:

| Hạng mục | Kết quả | Bằng chứng |
|---|---|---|
| Phân tích yêu cầu dự án | Hoàn thành | Yêu cầu FCAJ, cấu trúc workshop và tài liệu dự án |
| Phân tích source code | Hoàn thành | Các thư mục frontend, backend, chat, AI, database và cấu hình |
| Khởi tạo backend | Hoàn thành | Cấu trúc FastAPI, kết nối database, model và Alembic migration |
| Xây dựng xác thực | Hoàn thành | Đăng ký, đăng nhập, mã hóa mật khẩu, JWT và API được bảo vệ |
| Kiểm thử cục bộ | Hoàn thành một phần | Đã kiểm tra chức năng backend nhưng chưa lưu terminal output của Tuần 1 |
| Build frontend | Chưa yêu cầu | Tuần 1 chủ yếu tập trung phân tích hệ thống và khởi tạo backend |
| Triển khai AWS | Đã lên kế hoạch | Việc triển khai cloud được thực hiện trong các tuần tiếp theo |
| Giám sát chi phí AWS | Đã tìm hiểu và chuẩn bị | Tài liệu AWS Budgets và khái niệm cảnh báo ngân sách |
| Cấu hình IAM | Đã tìm hiểu và chuẩn bị | IAM user, role, policy và nguyên tắc cấp quyền tối thiểu |

### Kết quả tuần:

Đến cuối Tuần 1, tôi đã hiểu rõ hơn yêu cầu của chương trình thực tập, bài toán quản lý tuyển dụng, cấu trúc source code của ứng dụng và hướng triển khai AWS ban đầu.

Tôi đã xác định được các nhóm người dùng chính, module chức năng, dịch vụ runtime, quan hệ phụ thuộc dữ liệu và luồng xác thực. Tôi cũng có được kiến thức nền tảng về quản lý AWS Account, IAM, AWS Budgets và AWS Well-Architected Framework.

Mặc dù chưa có workload AWS nào được triển khai trong tuần này, phần phân tích đã tạo ra cơ sở kỹ thuật cần thiết cho các công việc container hóa, networking, database migration, Kubernetes, monitoring và CI/CD trong những tuần tiếp theo.

### Bài học rút ra:

Bài học chính trong Tuần 1 là không nên bắt đầu triển khai cloud trước khi hiểu rõ ranh giới và sự phụ thuộc giữa các thành phần ứng dụng.

Mặc dù dự án có thể trông giống một ứng dụng web duy nhất, frontend, API, chat thời gian thực, xử lý AI, cơ sở dữ liệu quan hệ, cache, lưu trữ chat và lưu trữ tài liệu có các yêu cầu triển khai và mở rộng khác nhau.

Tôi cũng nhận ra rằng việc thu thập bằng chứng cần được thực hiện song song với quá trình triển khai. Screenshot, kết quả test, commit hash, build log và deployment log sẽ dễ xác minh hơn nếu được lưu ngay khi thực hiện thay vì tổng hợp vào cuối dự án.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung củng cố nền tảng ứng dụng và dữ liệu trước khi triển khai cloud.

Các công việc dự kiến bao gồm xem xét database schema, cải thiện tính toàn vẹn dữ liệu, xử lý request trùng lặp và thao tác đồng thời, bổ sung cơ chế idempotency, mở rộng backend test và chuẩn bị các dịch vụ cho Docker containerization.

Quy trình thu thập bằng chứng cũng sẽ được cải thiện bằng cách lưu screenshot, kết quả test, commit reference và build output theo một cấu trúc thư mục thống nhất cho từng tuần.

### Kết quả đạt được trong tuần:

- Hiểu yêu cầu của chương trình thực tập và cấu trúc báo cáo FCAJ.
- Xác định được phạm vi project và hai nhóm người dùng chính là Candidate và HR/Company.
- Hoàn thiện sơ đồ tổng quan cho các chức năng authentication, jobs, applications, documents, chat và AI matching.
- Lựa chọn React/Vite, FastAPI, Node.js/Socket.IO và AI service cho kiến trúc ban đầu của hệ thống.
- Hiểu cách tạo AWS Account, thiết lập cảnh báo chi phí bằng AWS Budgets và quản lý quyền truy cập cơ bản bằng IAM.
- Khởi tạo được cấu trúc backend, database models, migration và authentication flow ban đầu.
<!--
TODO: Add screenshots, commits, test results, AWS Budget configuration, IAM configuration, or backend testing evidence for this week.
Expected image directory:
static/images/worklog/week-1/
-->