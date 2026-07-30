---
title: "Nhật ký tuần 2"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Tuần 2 - Phát triển chức năng Candidate-HR và lưu trữ tài liệu với Amazon S3

### Mục tiêu tuần 2:

- Phát triển các chức năng hồ sơ dành cho Candidate và HR/Company.
- Xây dựng các luồng đăng tuyển, tìm kiếm việc làm, nộp hồ sơ và quản lý ứng viên.
- Hoàn thiện cơ chế upload, kiểm tra và phân quyền truy cập tài liệu.
- Tìm hiểu Amazon S3, presigned URL và cách quản lý quyền truy cập tài nguyên bằng IAM.
- Làm quen với AWS CLI trong quá trình kiểm tra bucket và object storage.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Xây dựng Candidate profile và Company profile; bổ sung kiểm tra role, quyền sở hữu dữ liệu và các endpoint cập nhật thông tin hồ sơ. | 15/06/2026 | 15/06/2026 | [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html); [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) |
| 2 | Phát triển API tạo, cập nhật và quản lý job posting; xây dựng luồng tìm kiếm việc làm, nộp application và HR review applicant. | 16/06/2026 | 17/06/2026 | [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 3 | Tìm hiểu Amazon S3, cấu trúc bucket và object key; thực hành một số thao tác cơ bản bằng AWS CLI và cấu hình quyền truy cập tối thiểu bằng IAM. | 18/06/2026 | 18/06/2026 | [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html); [AWS CLI Command Reference for S3](https://docs.aws.amazon.com/cli/latest/reference/s3/); [IAM Policies and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) |
| 4 | Xây dựng document upload, file validation và permission checks; tích hợp S3 object key, presigned URL và kiểm thử tính nhất quán giữa database với object storage. | 19/06/2026 | 19/06/2026 | [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html); [Amazon S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html); [Uploading Objects with Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html) |

### Kết quả đạt được trong tuần:

- Hoàn thiện các chức năng hồ sơ cơ bản cho Candidate và HR/Company.
- HR có thể quản lý thông tin công ty, job posting và danh sách ứng viên.
- Candidate có thể tìm kiếm công việc, nộp application, tải tài liệu và theo dõi trạng thái hồ sơ.
- Các endpoint quan trọng đã được bổ sung kiểm tra role và quyền sở hữu dữ liệu.
- Hiểu cấu trúc bucket, object key và quyền truy cập cơ bản trong Amazon S3.
- Tích hợp luồng truy cập tài liệu riêng tư thông qua presigned URL.
- Kiểm thử được tính nhất quán giữa metadata lưu trong database và file lưu trên S3.

<!--
TODO: Add screenshots, commits, API test results, S3 bucket configuration, IAM policy, or document upload evidence for this week.
Expected image directory:
static/images/worklog/week-2/
-->