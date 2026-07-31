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

### Technical Implementation:

During Week 2, I focused on developing the core Candidate and HR/Company workflows while integrating document storage with Amazon S3.

For Candidate users, the system supports profile creation and updates, job browsing, job application submission, document upload, and application-status tracking. The Candidate flow was designed so that each user could manage personal information, view available opportunities, submit an application, attach supporting documents, and follow the progress of that application through the recruitment process.

For HR/Company users, the system supports company-profile management, job-posting creation, job updates, applicant review, and application-status management. HR users can maintain company information, publish job opportunities, review candidate profiles, inspect uploaded documents, and update the current state of an application.

A major part of the implementation was separating permissions between Candidate and HR/Company users. The backend does not rely only on authentication. It also checks the role of the current user before allowing access to protected business functions.

For example, Candidate users are allowed to submit applications but are not allowed to create or modify company job postings. HR users can manage job postings and applicants but cannot perform Candidate-only actions on behalf of another user.

Ownership checks were also applied to prevent users from accessing or modifying resources that do not belong to them. This was especially important for profiles, job postings, applications, and uploaded documents.

The role and ownership validation process helped reduce the risk of insecure direct object access. Even when a request contains a valid resource identifier, the backend still verifies whether the authenticated user is permitted to perform the requested action.

### Candidate and HR Workflow:

The Candidate and HR features were designed around the complete recruitment lifecycle.

The Candidate begins by creating or updating a personal profile. This information is later used when submitting applications and when HR users review the Candidate’s suitability for a job.

After browsing available job postings, the Candidate can submit an application. The backend validates the current user, confirms that the job exists, checks whether the application data is valid, and associates the application with both the Candidate and the selected job.

The Candidate can also upload a CV or other supporting documents. These documents are connected to the Candidate or application through metadata stored in the database.

On the HR side, the Company profile provides the business information associated with job postings. HR users can create new job records, update existing job descriptions, review lists of applicants, access authorized Candidate information, and update the progress of each application.

This workflow created a clear separation between the two user groups while still allowing them to interact through shared job and application data.

### Document Storage with Amazon S3:

The document-management feature was designed by separating file metadata from the actual file content.

The application database stores structured information such as the document owner, file name, content type, upload time, object key, and related application or profile. The actual binary file is stored as an object in Amazon S3.

This design prevents large files from being stored directly inside the relational database. PostgreSQL remains responsible for structured application data, while Amazon S3 provides scalable object storage for CVs, supporting documents, and other uploaded files.

Each uploaded file is identified by an S3 object key. The object key is generated or assigned by the backend and is stored together with the document metadata in the database.

The object-key structure helps organize files by owner, document type, or application context. It also reduces the risk of file-name collisions because the stored key does not need to depend only on the original file name.

The backend verifies that the database metadata and the stored S3 object refer to the same document. This relationship is important because the database controls ownership and business context, while S3 stores the file content itself.

### Presigned URL Integration:

The S3 bucket was designed to remain private instead of exposing uploaded documents through public URLs.

When an authorized user needs to access a private document, the backend generates a presigned URL. The URL grants temporary permission to access a specific S3 object without making the entire bucket or object publicly available.

This approach allows the application to maintain access control at the backend level. Before generating the URL, the system verifies the identity, role, and ownership permissions of the current user.

For example, a Candidate may access personal documents, while an HR user may access documents belonging to applicants for a job managed by that company. A user without the correct permission should not receive a valid presigned URL.

Presigned URLs also reduce the need for the backend to transfer large file contents directly. After authorization is completed, the browser can communicate with Amazon S3 using the temporary URL.

The URL is limited by an expiration time. After that period, the same link can no longer be used. This provides more control than a permanent public object URL.

### File Validation and Security:

The upload process includes file validation before the document is accepted.

The backend checks information such as the original file name, content type, file extension, and file size. These validations reduce the risk of unsupported or oversized files being stored.

The application does not depend only on the file name provided by the client. File names and extensions can be misleading, so validation must be performed using multiple request attributes and server-side rules.

The system also avoids exposing AWS credentials to the frontend. AWS access is handled by the backend or by temporary presigned URLs generated after authorization.

IAM permissions were reviewed according to the principle of least privilege. The application should only receive the S3 permissions required for its document operations, such as uploading, reading, or deleting objects within the intended bucket and path.

Keeping the bucket private, restricting IAM permissions, validating uploads, and checking ownership created multiple protection layers around document storage.

### AWS CLI Practice:

During this week, I also practiced basic AWS CLI operations related to Amazon S3.

The AWS CLI provided a direct way to verify the current AWS identity, inspect available buckets, list stored objects, and confirm the object-key structure used by the application.

Using command-line operations helped me understand the difference between application-level document metadata and storage-level S3 objects.

It also made it easier to verify whether an uploaded file had reached the correct bucket and whether the object key matched the value stored in the database.

This practice improved my understanding of how application code, AWS credentials, IAM permissions, and S3 resources interact during an upload request.

### Problems and Solutions:

| Problem | Root Cause | Resolution | Status |
|---|---|---|---|
| Candidate and HR users required different permissions. | Both user groups used the same backend but performed different business operations. | Role-based checks were added to restrict Candidate-only and HR-only functions. | Completed |
| Authentication alone did not prevent access to another user’s resource. | A valid user could still attempt to modify a resource by changing its identifier. | Ownership checks were added before profile, application, job, and document operations. | Completed |
| Public document URLs could expose private Candidate information. | Public S3 objects would bypass application-level authorization. | The S3 bucket was kept private and access was provided through presigned URLs. | Completed |
| Database records and S3 objects could become inconsistent. | File storage and metadata creation occur in separate systems. | The object key and document metadata were validated as part of the upload flow. | Completed |
| Uploaded files could be unsupported or too large. | Client-provided files cannot be trusted without validation. | File name, type, extension, and size checks were added before storage. | Completed |
| Broad S3 permissions could create unnecessary security risks. | Wide IAM policies are easier to configure but violate least privilege. | Permissions were limited to the required bucket and object actions. | Completed |

### Technical Knowledge Gained:

This week helped me understand that document upload is not simply a file-transfer operation.

A complete upload feature requires user authentication, role validation, ownership checks, file validation, object storage, metadata management, and controlled access.

I learned that Amazon S3 is designed for object storage rather than relational data. The database should describe the document and its relationship to the application, while S3 stores the actual file.

I also learned that an S3 object key is an internal storage identifier rather than a public link. The application can use this key to locate the object while still keeping the bucket private.

Presigned URLs showed how temporary access can be granted without exposing AWS credentials or changing the object to public.

Another important lesson was that authorization must be evaluated for every requested resource. A logged-in user should not automatically be allowed to access any profile, application, job, or file.

### Weekly Results:

By the end of Week 2, the application supported the main Candidate and HR/Company workflows required for recruitment management.

Candidates could manage their profiles, browse jobs, submit applications, upload supporting documents, and follow application progress.

HR/Company users could manage company information, create and update job postings, review applicant lists, access authorized Candidate information, and manage application status.

The project also completed its first practical AWS storage integration by connecting document management with Amazon S3.

Uploaded files were stored as private S3 objects, while the database maintained the related metadata and ownership information. Authorized access was provided through temporary presigned URLs.

This week strengthened both the business functionality and the security foundation of the application.

### Lessons Learned:

The main lesson from Week 2 was that authentication and authorization are different responsibilities.

Authentication confirms who the user is, while authorization determines what that user is allowed to do. A secure application needs both role validation and resource-ownership checks.

I also learned that cloud storage must be integrated with the application’s business rules. Amazon S3 stores the file, but the backend and database determine who owns the file, why it exists, and who is allowed to access it.

Using a private bucket with presigned URLs provided a safer design than exposing permanent public links.

The work also showed that storage consistency must be considered carefully because the database and S3 are separate systems. The application needs a clear relationship between document metadata and the corresponding object key.

### Next Week Plan:

The next week will focus on improving communication between services and preparing background-processing workflows.

The planned topics include asynchronous processing, message queues, retry handling, transactional outbox patterns, and reliable service-to-service communication.

The application services will also continue to be prepared for containerization and deployment in a scalable AWS environment.
<!--
TODO: Add screenshots, commits, API test results, S3 bucket configuration, IAM policy, or document upload evidence for this week.
Expected image directory:
static/images/worklog/week-2/
-->