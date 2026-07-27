---
title: "Nhật ký tuần 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Tuần 2 - Nền tảng Candidate-HR, applications, documents và S3

### Mục tiêu tuần 2:

- Phát triển các chức năng cho Candidate và HR/Company.
- Hoàn thiện job posting, job browsing, application submission và applicant review flows.
- Hỗ trợ document upload, validation, access control và Amazon S3 storage.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Xây dựng Candidate profile và Company profile. | 15/06/2026 | 21/06/2026 | [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html); [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) |
| 2 | Xây dựng API tạo, cập nhật, xem và quản lý jobs. | 15/06/2026 | 21/06/2026 | [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 3 | Xây dựng application submission, HR dashboard và applicant review. | 15/06/2026 | 21/06/2026 | [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html); [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) |
| 4 | Xây dựng document upload, listing, file validation và permission checks. | 15/06/2026 | 21/06/2026 | [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html); [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) |
| 5 | Tích hợp S3 object keys, presigned URLs và kiểm thử database-storage consistency. | 15/06/2026 | 21/06/2026 | [Amazon S3 User Guide - Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html); [Amazon S3 - Uploading Objects with Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html) |

### Kết quả đạt được trong tuần:

- Hệ thống hỗ trợ cả vai trò Candidate và HR.
- HR users có thể quản lý company information, jobs và applicants.
- Candidates có thể xem jobs, nộp applications, upload documents và theo dõi trạng thái.
- Access control kiểm tra role và object ownership.
- Document flows hỗ trợ validation và truy cập S3 private thông qua presigned URLs.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-2/
-->
