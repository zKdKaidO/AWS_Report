---
title: "Week 3 Worklog"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Week 3 - React Frontend, REST API Integration, and Static Deployment on AWS

### Week 3 Objectives:

- Build the main interfaces for Candidate and HR/Company users.
- Integrate the React frontend with backend REST APIs.
- Complete the authentication context, role-based routing, and protected pages.
- Prepare the production build and configure environment variables and CORS.
- Learn how to host a static website with Amazon S3 and distribute content through Amazon CloudFront.

### Tasks Carried Out This Week:

| Day | Task | Start Date | Completion Date | Reference Material |
|---|---|---|---|---|
| 1 | Initialized the React application with Vite, configured Tailwind CSS, organized the frontend folder structure, and created reusable interface components. | 22/06/2026 | 22/06/2026 | [React Documentation](https://react.dev/learn); [Vite Getting Started](https://vite.dev/guide/); [Tailwind CSS with Vite](https://tailwindcss.com/docs/installation/using-vite) |
| 2 | Developed the registration and login pages, authentication context, user-session handling, and role-based navigation for Candidate and HR/Company users. | 23/06/2026 | 23/06/2026 | [React Documentation](https://react.dev/learn); [React Router Documentation](https://reactrouter.com/) |
| 3 | Developed the Candidate dashboard, job board, profile, application detail, and document management pages, and integrated the related jobs, applications, and documents APIs. | 24/06/2026 | 24/06/2026 | [React Documentation](https://react.dev/learn); [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) |
| 4 | Developed the HR dashboard, company profile, job editor, and applicant review pages; added loading states, error handling, and access checks for protected pages. | 25/06/2026 | 25/06/2026 | [React Documentation](https://react.dev/learn); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 5 | Prepared the production build, configured environment variables, and tested CORS behavior; studied static website hosting with Amazon S3 and content delivery through Amazon CloudFront. | 26/06/2026 | 26/06/2026 | [Vite - Building for Production](https://vite.dev/guide/build); [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS); [Hosting a Static Website with Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html); [Amazon CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) |

### Week 3 Achievements:

- Completed the main interfaces for Candidate and HR/Company users.
- Enabled the frontend to call authentication, jobs, applications, documents, and dashboard APIs.
- Implemented role-based navigation after login.
- Protected private pages using authentication state and role checks.
- Added loading states, API error handling, and invalid-session handling.
- Generated and validated the frontend production build.
- Understood how Amazon S3 can host a static frontend and how CloudFront can improve content delivery performance.

<!--
TODO: Add frontend screenshots, commits, API integration tests, production build output, S3 static hosting configuration, or CloudFront evidence for this week.
Expected image directory:
static/images/worklog/week-3/
-->