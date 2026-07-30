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