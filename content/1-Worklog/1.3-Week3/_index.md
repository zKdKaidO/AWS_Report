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

### Technical Implementation:

During Week 3, I developed the React frontend for Candidate and HR/Company users and connected it to the backend REST APIs.

The frontend was initialized with React and Vite, while Tailwind CSS was used to build consistent layouts, forms, cards, tables, and navigation components. The source code was separated into pages, reusable components, authentication context, routing, and API service modules.

<pre>
Web Browser
    |
    v
React Frontend
    |
    +-- Authentication Context
    +-- React Router
    +-- Candidate Pages
    +-- HR/Company Pages
    +-- API Service Layer
    |
    v
Backend REST APIs
</pre>

### Authentication and Role-Based Routing:

The login and registration pages were integrated with the backend authentication APIs. After login, the application stores the current user state and redirects the user according to their role.

Protected routes verify both authentication status and user role. Candidate users are directed to Candidate features, while HR/Company users can access company and recruitment-management pages.

<pre>
Login Form
    |
    v
Authentication API
    |
    +-- Valid session
    |      |
    |      v
    |  Load user role
    |      |
    |      v
    |  Redirect to dashboard
    |
    +-- Invalid session
           |
           v
       Show error
</pre>

Frontend route protection improves usability, but backend authorization remains responsible for enforcing actual access permissions.

### Candidate Features:

Candidate users can manage their profiles, browse available jobs, view job details, submit applications, upload supporting documents, and track application status.

The Candidate pages communicate with the jobs, profiles, applications, and documents APIs through reusable service functions.

<pre>
Candidate Dashboard
    |
    +-- Manage profile
    +-- Browse jobs
    +-- View job details
    +-- Submit application
    +-- Upload documents
    +-- Track application status
</pre>

### HR and Company Features:

HR/Company users can manage company information, create and update job postings, view applicant lists, review Candidate details, access authorized documents, and update application status.

<pre>
HR Dashboard
    |
    +-- Manage company profile
    +-- Create job posting
    +-- Update job posting
    +-- View applicants
    +-- Review documents
    +-- Update application status
</pre>

### REST API Integration:

API requests were organized into a separate service layer instead of being written directly inside every page component.

This structure keeps endpoint paths, request headers, payloads, authentication handling, and error processing consistent across the frontend.

<pre>
React Page
    |
    v
API Service Function
    |
    +-- Build request
    +-- Add authentication data
    +-- Send request
    +-- Parse response
    |
    v
Backend REST API
    |
    v
Update React State
</pre>

Loading states and error messages were added to prevent incomplete data from being displayed while requests were still running. Invalid sessions are cleared and redirected to the login page.

### Production Build and Environment Configuration:

The backend API URL was configured through Vite environment variables so that development and production environments could use different endpoints.

The production build converts the React project into optimized static HTML, JavaScript, CSS, and asset files.

<pre>
React Source Code
        |
        v
Vite Production Build
        |
        +-- Bundle JavaScript
        +-- Process CSS
        +-- Resolve assets
        +-- Apply environment values
        |
        v
Static Build Files
</pre>

CORS configuration was also reviewed because the frontend and backend may run on different domains. Production CORS settings should allow only trusted frontend origins.

### Amazon S3 and CloudFront:

Amazon S3 was studied as the storage location for the static frontend build. Because the production output contains static files, it can be hosted without maintaining a separate frontend application server.

Amazon CloudFront can distribute and cache these files through edge locations, improving delivery performance and providing HTTPS support.

<pre>
User Browser
      |
      v
Amazon CloudFront
      |
      +-- Cached file available
      |         |
      |         v
      |    Return cached file
      |
      +-- File not cached
                |
                v
          Amazon S3 Bucket
                |
                v
          Return static file
</pre>

For React Router, CloudFront and S3 must return `index.html` for frontend routes such as `/jobs` or `/dashboard`. Otherwise, refreshing a page may result in a not-found error.

### Problems and Solutions:

| Problem | Resolution | Status |
|---|---|---|
| Candidate and HR users required different interfaces. | Added role-based navigation and protected routes. | Completed |
| API logic was repeated across pages. | Created reusable API service modules. | Completed |
| Pages rendered before API data was available. | Added loading states and conditional rendering. | Completed |
| Failed API requests provided unclear feedback. | Added error messages and invalid-session handling. | Completed |
| Development and production used different API URLs. | Moved the backend URL to Vite environment configuration. | Completed |
| Cross-origin API requests could be blocked. | Reviewed and configured CORS for trusted frontend origins. | Completed |
| React routes could fail after page refresh. | Planned an `index.html` fallback for S3 and CloudFront. | Completed |

### Technical Knowledge Gained:

This week helped me understand how frontend responsibilities should be separated between components, routing, authentication, state management, and API services.

I also learned that frontend role checks improve navigation but cannot replace backend authorization.

The production build demonstrated how React source code becomes static files that can be hosted using Amazon S3 and distributed through Amazon CloudFront.

### Weekly Results:

By the end of Week 3, the main Candidate and HR/Company interfaces were completed and integrated with the backend APIs.

Authentication context, protected routing, loading states, error handling, environment configuration, and production build preparation were also completed.

The planned deployment architecture uses Amazon S3 to store the static frontend and Amazon CloudFront to distribute it securely and efficiently.

### Lessons Learned:

A complete frontend requires more than interface design. It must also handle authentication, routing, API communication, loading states, errors, environment configuration, and deployment behavior.

Amazon S3 and CloudFront provide complementary roles: S3 stores the static files, while CloudFront caches and delivers them to users.

### Next Week Plan:

The next week will focus on containerization, repeatable builds, automated testing, security checks, and preparation for deployment in an AWS runtime environment.

<!--
TODO: Add frontend screenshots, commits, API integration tests, production build output, S3 static hosting configuration, or CloudFront evidence for this week.
Expected image directory:
static/images/worklog/week-3/
-->