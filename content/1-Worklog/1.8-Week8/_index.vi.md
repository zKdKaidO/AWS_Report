---
title: "Nhật ký tuần 8"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Tuần 8 - CI/CD, chuẩn bị triển khai AWS, testing và báo cáo

### Mục tiêu tuần 8:

- Tự động hóa build, test, security scanning và deployment preparation.
- Chuẩn bị và rà soát production architecture trên AWS.
- Kiểm thử toàn bộ Candidate-HR flow và hoàn thiện báo cáo project.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Hoàn thiện GitHub Actions cho backend, frontend, chat, infrastructure và smoke tests. | 27/07/2026 | 30/07/2026 | [GitHub Actions Documentation - OpenID Connect with AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws) |
| 2 | Bổ sung Gitleaks, Trivy, pre-commit và kiểm tra secret/IAM/CORS. | 27/07/2026 | 30/07/2026 | [OWASP Secure Code Review Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Code_Review_Cheat_Sheet.html); [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |
| 3 | Chuẩn bị ECR, EKS, ALB, OIDC và frontend deployment S3/CloudFront. | 27/07/2026 | 30/07/2026 | [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html); [GitHub Actions Documentation - OpenID Connect with AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws) |
| 4 | Chạy smoke test và end-to-end test cho auth, jobs, applications, documents, AI và chat. | 27/07/2026 | 30/07/2026 | [OWASP Secure Code Review Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Code_Review_Cheat_Sheet.html); [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |
| 5 | Hoàn thiện README, runbooks, architecture documentation và báo cáo project. | 27/07/2026 | 30/07/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html); [AWS Builders' Library - Timeouts, Retries and Backoff with Jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) |

### Kết quả đạt được trong tuần:

- CI/CD pipelines được chuẩn bị cho nhiều services.
- Automated tests và security scans được cấu hình trước deployment.
- Production path sử dụng ECR, EKS, ALB, RDS, DynamoDB, ElastiCache, S3 và CloudFront.
- Candidate và HR flows được chuẩn bị cho end-to-end validation.
- Technical documentation và report content được hoàn thiện để nộp.

<!--
Evidence required: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-8/
-->
