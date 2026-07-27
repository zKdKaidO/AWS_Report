---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Week 2 - Candidate-HR Platform, Applications, Documents, and S3

### Week 2 Objectives:

- Develop Candidate and HR/Company platform features.
- Complete job posting, job browsing, application submission, and applicant review flows.
- Support document upload, validation, access control, and Amazon S3 storage.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Build Candidate profile and Company profile features. | 15/06/2026 | 21/06/2026 | [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html); [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) |
| 2 | Build APIs to create, update, browse, and manage jobs. | 15/06/2026 | 21/06/2026 | [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 3 | Build application submission, HR dashboard, and applicant review features. | 15/06/2026 | 21/06/2026 | [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html); [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) |
| 4 | Build document upload, listing, file validation, and permission checks. | 15/06/2026 | 21/06/2026 | [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html); [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) |
| 5 | Integrate S3 object keys, presigned URLs, and database-storage consistency tests. | 15/06/2026 | 21/06/2026 | [Amazon S3 User Guide - Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html); [Amazon S3 - Uploading Objects with Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html) |

### Week 2 Achievements:

- The system supports both Candidate and HR roles.
- HR users can manage company information, jobs, and applicants.
- Candidates can view jobs, submit applications, upload documents, and track status.
- Access control checks role and object ownership.
- Document flows support validation and private S3 access through presigned URLs.

<!--
TODO: Add screenshots, commits, test results, or deployment evidence for this week.
Expected image directory:
static/images/worklog/week-2/
-->
