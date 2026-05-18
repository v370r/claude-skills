---
name: linkedin-apply
description: LinkedIn job application assistant for the /linkedin-apply trigger or requests to search recent LinkedIn Easy Apply roles, submit Easy Apply applications for jobs with at least 40% fit or at least two matching role signals, optionally contact listed recruiters through free messaging or connection flows without spending InMail credits, and report outcomes.
---

# LinkedIn Apply

## Profile

> **SETUP REQUIRED:** Replace all bracketed placeholders below with your own information before using this skill.

- [Degree], [University], [GPA], graduating [Month Year]
- [X years] [Job Title] at [Company]: [key technologies and accomplishments]
- [Additional experience]: [role, company, technologies]
- Projects: [project names, key technologies]
- Target roles: [list the job titles you want to apply to]
- Preferred roles: [remote, hybrid, on-site, preferred locations]

## Operating Rules

- Use the browser connected to the user's LinkedIn session. Prefer the Chrome profile that has the user's active LinkedIn session.
- Do not bypass CAPTCHA, paywalls, account security, or LinkedIn restrictions. Pause and report the blocker if encountered.
- Do not spend InMail credits.
- Default to at most 10 submitted applications per run unless the user gives a different limit.
- Submit only when the user's request clearly invokes this applying workflow, such as `/linkedin-apply`.
- Do not fabricate answers. If a required field cannot be answered from the profile, visible resume, LinkedIn defaults, or prior user-provided context, pause and ask the user before submitting that job.
- For voluntary demographic, disability, veteran, and self-identification fields, choose `Decline to answer` or equivalent when available unless the user has provided a preference.
- **NO Python/bash scripts** for parsing Playwright snapshot JSON files or LinkedIn page data. Use built-in tools only: `Grep` for searching snapshot text, `Read` for reading files, `browser_evaluate` for DOM queries. Only use scripting if the user explicitly requests it.

## Search

Open LinkedIn Jobs and search each query:

1. `[Target Role 1]` (e.g., Backend Software Engineer)
2. `[Target Role 2]` (e.g., Full Stack Engineer)
3. `[Target Role 3]` (e.g., Software Engineer)
4. `[Target Role 4]` (e.g., Python Software Engineer)

Apply these filters using the LinkedIn filter UI controls (dropdowns, checkboxes, buttons) — do not use URL parameters to set filters, as LinkedIn may reject them.

**If filters fail to load or show "There was a problem loading your filters":**
1. Click the **Search** button (the search input is already populated) — this often triggers a proper reload and fixes the filter panel.
2. If that doesn't work, navigate to `https://www.linkedin.com/jobs/` first (the plain jobs landing page), wait for it to fully load, then navigate to the search URL again.
3. As a fallback, use a URL that already includes filter parameters (e.g., `f_AL=true` for Easy Apply, `f_TPR=r86400` for past 24 hours, `f_WT=2` for Remote, `f_E=2,3` for Mid-Senior) — the URL itself acts as the filter, and the search results page should render even if the filter sidebar shows an error.
4. If "No matching jobs" appears but results exist, ignore the error and scroll the results list — the jobs are often still rendered despite the filter panel error.

**HARD STOP — do NOT proceed with applying if filters cannot be loaded:**
- After trying all recovery steps above, if the filter panel still shows an error, stop and report to the user. Do NOT apply to jobs from unfiltered results — the Easy Apply, date posted, and experience level filters are critical to the workflow.

Apply these filter settings:

- Date posted: Past 24 hours
- Easy Apply: On
- Experience level: Mid-Senior (or appropriate for user's experience)
- Location: Remote or [user's preferred locations]
- Sort: most recent when available

Deduplicate jobs by LinkedIn job ID or canonical job URL.

## Match

Apply to Easy Apply jobs when either condition is true:

- The visible LinkedIn match score or estimated resume/profile fit is at least 40%.
- At least two relevant role signals match, such as title, required responsibilities matching the user's skills, or key technologies from the user's background.

Treat 40% fit or two matching role signals as sufficient for submission; do not wait for a stronger match when the job passes the hard rules below.

Hard rules:

- Must show Easy Apply and must not redirect to an external site for application submission.
- Must be posted within the past 24 hours.
- Must be in user's preferred locations (remote, hybrid, or on-site as configured).
- Must align with the user's target roles and skill set.
- Experience requirement should be compatible with the user's years of experience.
- Skip jobs requiring significantly more experience than the user has (e.g., 7+ years if user has < 5).
- Skip roles focused primarily on areas outside the user's expertise (e.g., ML/data science for a backend engineer, unless the user has those skills).

Prefer roles mentioning technologies and skills from the user's background, but do not skip an otherwise valid Easy Apply role only because it is a moderate match.

## Apply

For each selected job:

1. Open the job posting.
2. Confirm it is still Easy Apply and still within the past 24 hours.
3. Click `Easy Apply`.
4. Fill required fields from the profile, LinkedIn defaults, and resume information.
5. If resume upload is prompted, use the resume already selected by LinkedIn when appropriate; otherwise locate the user's current resume only if a clear resume file is available. Ask before uploading if multiple candidate resumes are ambiguous.
6. For application location fields, use the job posting location or `[Default Location]` for remote roles.
7. For work authorization questions, use the user-provided defaults from the profile section. If not configured, pause and ask the user.
8. For compensation, start date, relocation, background check, or other legal/preference questions not already known, pause and ask the user rather than guessing.
9. For `Why are you interested?`, cover letter, or similar open text fields, write 3-4 concise sentences tailored to the job. Match keywords from the posting to the user's stack and target roles listed in the Profile section. Keep the tone direct and professional.
10. Review required fields and submit the application.
11. Record the company, role, URL, application result, and any skipped reason.

## Recruiter Outreach

After submitting an application, inspect the same job posting for `Meet the hiring team` or a visible recruiter/hiring manager.

- If no hiring team section is visible, skip outreach.
- If a recruiter or hiring manager is listed, open the listed profile from the job posting.
- First verify the free direct messaging path: if a free `Message` action is available and does not open paid InMail, Premium, or a credit prompt, send the note below.
- If `Message` opens paid InMail, Premium, or a credit prompt, do not send it and do not continue through payment or credits. This is not a final skip result.
- After any paid/Premium `Message` path, explicitly verify the free connection request path by looking for `Connect`, including under `More` if needed.
- If `Connect` allows `Add a note`, send the same note as the connection note. If LinkedIn's note character limit blocks the exact text, shorten only enough to fit while preserving the name, job title, key experience highlights, and connect request.
- If `Connect` is free but does not offer `Add a note`, send the default connection request without a note instead of skipping, then record `Connection sent without note`.
- Only skip outreach after both the free direct `Message` path and the free `Connect` request path have been verified and neither can be completed without paid InMail, Premium, credits, or a LinkedIn restriction. Do not report `InMail required` as the final reason unless the `Connect` fallback was attempted and failed.

Use this note template, filling in `[Name]`, `[Job Title]`, and the user's experience details:

```text
Hi [Name], I just applied for the [Job Title] role and wanted to reach out directly. I'm a [your role summary, e.g., backend/full-stack engineer] with [X years] at [Company] building [technologies/achievements]. Would love to connect if you have a moment. Thanks!
```

## Report

Return a concise summary table:

| Company | Role | Applied? | Recruiter Message Sent? |
| --- | --- | --- | --- |

Use `Yes`, `No`, or `Skipped: reason` in the final two columns. Include a short note below the table for any blocker that prevented completion.
