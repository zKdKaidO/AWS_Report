---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Week 8 - CI/CD, AWS Deployment Preparation, Testing, and Reporting

### Week 8 Objectives:

- Automate build, test, security scanning, and deployment preparation.
- Prepare and review the production architecture on AWS.
- Test the full Candidate-HR flow and complete the final project report.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Complete GitHub Actions for backend, frontend, chat, infrastructure, and smoke tests. | 27/07/2026 | 30/07/2026 | [GitHub Actions Documentation - OpenID Connect with AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws) |
| 2 | Add Gitleaks, Trivy, pre-commit, and secret/IAM/CORS checks. | 27/07/2026 | 30/07/2026 | [OWASP Secure Code Review Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Code_Review_Cheat_Sheet.html); [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |
| 3 | Prepare ECR, EKS, ALB, OIDC, and S3/CloudFront frontend deployment path. | 27/07/2026 | 30/07/2026 | [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html); [GitHub Actions Documentation - OpenID Connect with AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws) |
| 4 | Run smoke and end-to-end tests for auth, jobs, applications, documents, AI, and chat. | 27/07/2026 | 30/07/2026 | [OWASP Secure Code Review Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Code_Review_Cheat_Sheet.html); [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |
| 5 | Complete README, runbooks, architecture documentation, and project report content. | 27/07/2026 | 30/07/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html); [AWS Builders' Library - Timeouts, Retries and Backoff with Jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) |

### Week 8 Achievements:

- CI/CD pipelines are prepared for multiple services.
- Automated tests and security scans are configured before deployment.
- The production path uses ECR, EKS, ALB, RDS, DynamoDB, ElastiCache, S3, and CloudFront.
- Candidate and HR flows are prepared for end-to-end validation.
- Technical documentation and report content are completed for submission.

<!--
Evidence required: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-8/
-->
