---
title: "Week 2 Worklog"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Week 2 - Candidate-HR Features and Document Storage with Amazon S3

### Week 2 Objectives:

- Develop profile features for Candidate and HR/Company users.
- Build job posting, job searching, application submission, and applicant management flows.
- Implement document upload, validation, and access control.
- Learn the fundamentals of Amazon S3, presigned URLs, and resource permissions with IAM.
- Become familiar with basic AWS CLI operations for checking buckets and stored objects.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Developed Candidate and Company profiles, including role validation, data ownership checks, and profile update endpoints. | 15/06/2026 | 15/06/2026 | [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html); [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) |
| 2 | Developed APIs for creating, updating, and managing job postings, together with job searching, application submission, and HR applicant review flows. | 16/06/2026 | 17/06/2026 | [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 3 | Studied Amazon S3 bucket and object-key structures, practiced basic operations with AWS CLI, and reviewed least-privilege access using IAM policies. | 18/06/2026 | 18/06/2026 | [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html); [AWS CLI Command Reference for S3](https://docs.aws.amazon.com/cli/latest/reference/s3/); [IAM Policies and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) |
| 4 | Implemented document upload, file validation, and permission checks; integrated S3 object keys and presigned URLs; and tested consistency between database records and object storage. | 19/06/2026 | 19/06/2026 | [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html); [Amazon S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html); [Uploading Objects with Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html) |

### Week 2 Achievements:

- Completed the basic profile features for Candidate and HR/Company users.
- Enabled HR users to manage company information, job postings, and applicant lists.
- Enabled Candidates to browse jobs, submit applications, upload documents, and track application status.
- Added role and object-ownership checks to important endpoints.
- Understood the basic structure of Amazon S3 buckets, object keys, and access permissions.
- Integrated secure access to private documents through presigned URLs.
- Verified consistency between metadata stored in the database and files stored in Amazon S3.

<!--
TODO: Add screenshots, commits, API test results, S3 bucket configuration, IAM policy, or document upload evidence for this week.
Expected image directory:
static/images/worklog/week-2/
-->