---
name: linkedin-apply
description: LinkedIn job application assistant for the /linkedin-apply trigger or requests to search recent LinkedIn Easy Apply software engineering roles, submit Easy Apply applications for jobs with at least 40% fit or at least two matching role signals, optionally contact listed recruiters through free messaging or connection flows without spending InMail credits, and report outcomes.
---

# LinkedIn Apply

## Profile

Use this background unless the user overrides it:

- MS Computer Science, University of Colorado Boulder, GPA 3.9, graduating May 2026
- 4 years software engineering experience, including 3 years Senior Software Engineer at ADP: Java, Spring Boot, Kafka, microservices, Angular, React, monolith migrations
- Graduate co-op at Aragorn Racing Corp: AWS Lambda, SurrealDB, serverless backend
- Projects: pWin.ai, RAG pipeline, LangChain, SurrealDB; GetMyURI, founding engineer
- Target roles: Backend Engineer, Full-Stack Engineer, Software Engineer, Python Software Engineer
- Preferred roles: remote or hybrid, US-based

## Operating Rules

- Use the browser connected to the user's LinkedIn session. Prefer Chrome Profile 7 when browser profile choice is needed.
- Do not bypass CAPTCHA, paywalls, account security, or LinkedIn restrictions. Pause and report the blocker if encountered.
- Do not spend InMail credits.
- Do not paste or store Jobright cookies, `SESSION_ID`, authorization tokens, or other session secrets in this skill. Use the active browser session or local environment only.
- Do not send email to guessed addresses. Only email a recruiter or hiring-team contact when Jobright or another visible source returns a concrete email address for that person.
- Do not mention AI, bots, automation, assistant names, or helper signatures in LinkedIn notes or emails unless the user explicitly asks for that in the current session.
- Default to at most 10 submitted applications per run unless the user gives a different limit.
- Submit only when the user's request clearly invokes this applying workflow, such as `/linkedin-apply`.
- Do not fabricate answers. If a required field cannot be answered from the profile, visible resume, LinkedIn defaults, or prior user-provided context, pause and ask the user before submitting that job.
- For voluntary demographic, disability, veteran, and self-identification fields, choose `Decline to answer` or equivalent when available unless the user has provided a preference.

## Search

Open LinkedIn Jobs and search each query:

1. `Backend Software Engineer`
2. `Full Stack Engineer`
3. `Software Engineer`
4. `Python Software Engineer`

Apply these filters:

- Date posted: Past 24 hours
- Easy Apply: On
- Experience level: Mid-Senior
- Location: Remote or United States
- Sort: most recent when available

Deduplicate jobs by LinkedIn job ID or canonical job URL.

## Match

Apply to Easy Apply jobs when either condition is true:

- The visible LinkedIn match score or estimated resume/profile fit is at least 40%.
- At least two relevant role signals match, such as title, required backend/full-stack responsibilities, Java, Spring Boot, Kafka, AWS, Python, distributed systems, microservices, APIs, React, Angular, serverless, databases, or backend platform work.

Treat 40% fit or two matching role signals as sufficient for submission; do not wait for a stronger match when the job passes the hard rules below.

Hard rules:

- Must show Easy Apply and must not redirect to an external site for application submission.
- Must be posted within the past 24 hours.
- Must be US-based, remote, or hybrid.
- Must align with backend, full-stack, Python, or general software engineering.
- Experience requirement should be 3-5 years, unspecified, or otherwise clearly compatible with the user's background.
- Java exception: for Java, Spring Boot, backend Java, distributed systems, microservices, or Java full-stack roles, apply even when the title is Senior, Staff, Lead, or similar and the posting asks for up to 10 years of experience, as long as the role has Easy Apply and at least two strong matching Java/backend signals.
- Skip jobs requiring 7+ years only when they do not qualify for the Java exception above.
- Skip roles focused primarily on ML/data science, iOS, Android, mobile-only development, QA-only, support-only, sales, clinical, or non-engineering work.

Prefer roles mentioning Java, Spring Boot, Kafka, AWS, Python, distributed systems, microservices, APIs, React, Angular, serverless, databases, or backend platform work, but do not skip an otherwise valid Easy Apply role only because it is a moderate match.

## Apply

For each selected job:

1. Open the job posting.
2. Confirm it is still Easy Apply and still within the past 24 hours.
3. Click `Easy Apply`.
4. Fill required fields from the profile, LinkedIn defaults, and resume information.
5. If resume upload is prompted, use the resume already selected by LinkedIn when appropriate; otherwise locate the user's current resume only if a clear resume file is available. Ask before uploading if multiple candidate resumes are ambiguous.
6. For application location fields, use the job posting location or `United States` for remote roles. Do not enter Boulder, Colorado unless the job posting itself is Boulder-based or the user explicitly says to.
7. For work authorization questions, use these user-provided defaults unless the user overrides them:
   - `Are you legally authorized to work in the US?` -> `Yes`.
   - `Would you require visa sponsorship now or in the future?` -> `Yes`.
   - For free-text work authorization or sponsorship explanation fields, say: `Yes, I would require sponsorship in the future if needed.`
8. For compensation, relocation, background check, or other legal/preference questions not already known, use these user-provided defaults unless the user overrides them:
   - Compensation minimum -> `45 USD/hour`
   - Relocation -> `Yes, willing to relocate`
   - Background check -> `Yes`
   - `Can you start immediately?` -> `Yes`
   - Certification/license questions for credentials the user has not explicitly claimed -> `No`
9. For `Why are you interested?`, cover letter, or similar open text fields, write 3-4 concise sentences tailored to the job. Match keywords from the posting to the user's stack and target roles: Java, Spring Boot, Kafka, AWS Lambda, Python, microservices, React, Angular, SurrealDB, LangChain, RAG, serverless backend, distributed systems, and monolith migrations. Keep the tone direct and professional.
10. Review required fields and submit the application.
11. Record the company, role, URL, application result, and any skipped reason.

## Recruiter Outreach

After submitting an application, inspect the same job posting for `Meet the hiring team` or a visible recruiter/hiring manager.

- If no hiring team section is visible, skip outreach.
- If a recruiter or hiring manager is listed, open the listed profile from the job posting.
- First verify the free direct messaging path: if a free `Message` action is available and does not open paid InMail, Premium, or a credit prompt, send the note below.
- If `Message` opens paid InMail, Premium, or a credit prompt, do not send it and do not continue through payment or credits. This is not a final skip result.
- After any paid/Premium `Message` path, explicitly verify the free connection request path by looking for `Connect`, including under `More` if needed.
- If `Connect` allows `Add a note`, send the same note as the connection note. If LinkedIn's note character limit blocks the exact text, shorten only enough to fit while preserving the name, job title, backend/full-stack profile, Java/Spring Boot experience, current contract role, and connect request.
- If `Connect` is free but does not offer `Add a note`, send the default connection request without a note instead of skipping, then record `Connection sent without note`.
- Only skip outreach after both the free direct `Message` path and the free `Connect` request path have been verified and neither can be completed without paid InMail, Premium, credits, or a LinkedIn restriction. Do not report `InMail required` as the final reason unless the `Connect` fallback was attempted and failed.

Use this note exactly except fill `[Name]` and `[Job Title]`:

```text
Hi [Name], I just applied for the [Job Title] role and wanted to reach out directly. I'm a backend/full-stack engineer with 3+ years at ADP building microservices with Java and Spring Boot, and currently working on contract as senior dev at Aragorn racing corporation. Would love to connect if you have a moment. Thanks!
```

## Jobright Email Follow-Up

After the LinkedIn recruiter or hiring-team outreach step, try to find a work email through Jobright for each recruiter or hiring-team LinkedIn profile URL that was found.

Preferred API flow:

1. Use the active Jobright browser session or a local-only `SESSION_ID` environment variable; never write the session value into the skill or final report.
2. Request `GET https://jobright.ai/swan/email/external-linkedin-to-email?url=[encoded LinkedIn profile URL]`.
3. Send the same-origin-style headers Jobright expects when using a direct request: `accept: application/json, text/plain, */*`, `referer: https://jobright.ai/jobs/info/[jobright job id]`, and `x-client-type: web`. Include the Jobright session cookie only from local browser/session context.
4. Treat a successful response as valid only when JSON has `success: true`, `errorCode: 10000`, and `result.emails` contains at least one concrete email address.
5. Use `result.userInfo.name`, `result.userInfo.jobTitle`, and `result.userInfo.companyName` as helper context, but prefer the LinkedIn/job posting name, role, and company when they are clearer.
6. Before sending each email, show the parsed sequence item to the user or in the run log: contact name, LinkedIn profile URL, parsed email, company, role, and planned Gmail sender.

Browser fallback flow:

1. Open Jobright.ai in the next browser tab. If a Jobright job URL is already available, use that job page. Otherwise search Jobright by company and job title and open the matching job page when the match is clear.
2. On the Jobright job page, find the `Find Any Email` panel.
3. Paste the LinkedIn profile URL into the textbox with placeholder text like `Paste any LinkedIn profile URL...`, then click the `find-email` search button.
4. If Jobright asks to `Connect Now` before revealing the email, click `Connect Now` only when it is free and does not require payment, credits, or a plan upgrade.
5. If Jobright does not reveal an email, requires payment/credits, or shows a restriction, do not guess. Record `Skipped: no free Jobright email`.

When an email is found, send from the connected Gmail account `vijaykumarpolojuofficial@gmail.com`. Do not attach a resume by default. Point the recipient to the user's portfolio, `https://vijaypoloju.tech`, as the professional place to review the resume, projects, and background. Attach a resume only when the user explicitly asks for an attachment in the current session and the resume path is unambiguous.

Use a natural, non-AI-obvious email in this format except fill `[Name]`, `[Job Title]`, and `[Company]`:

```text
Hi [Name],

I hope you're doing well.

I came across the [Job Title] role at [Company] and it looks like a strong match for my background. I recently applied and wanted to reach out directly.

I'm a backend/full-stack engineer with 4 years of software engineering experience across Java, Spring Boot, microservices, cloud backend work, and full-stack delivery. I try to be the kind of honest, dependable all-rounder who can pick up messy problems and keep moving. My portfolio is here if helpful: vijaypoloju.tech

Could you share any advice about the role or team? My portfolio has my resume, projects, and background in one place: https://vijaypoloju.tech. I would really appreciate it if you could forward it to the hiring team if it seems relevant.

Thank you,
VIJAY KUMAR POLOJU
```

## Report

Return a concise summary table:

| Company | Role | Applied? | Recruiter Message Sent? | Jobright Email Sent? |
| --- | --- | --- | --- | --- |

Use `Yes`, `No`, or `Skipped: reason` in the final outreach columns. Include a short note below the table for any blocker that prevented completion.
