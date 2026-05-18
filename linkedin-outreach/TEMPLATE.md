---
name: linkedin-outreach
description: >
  Automate LinkedIn recruiter outreach and message follow-ups using Playwright MCP browser automation.
  Use this skill whenever the user wants to: find and message recruiters on LinkedIn, send personalized
  connection requests or InMail to recruiters hiring for their roles, follow up on unanswered LinkedIn
  messages, bulk-connect with recruiters, do LinkedIn outreach, automate LinkedIn networking, or check
  LinkedIn messages and send follow-ups. Triggers on phrases like "reach out to recruiters", "LinkedIn
  outreach", "connect with recruiters", "follow up on LinkedIn", "message recruiters", "LinkedIn automation",
  "send connection requests", "InMail recruiters", or any mention of automating LinkedIn communication.
  Also triggers when the user mentions Playwright MCP in the context of LinkedIn.
compatibility: Requires Playwright MCP server connected for browser automation
---

# LinkedIn Recruiter Outreach & Follow-Up Skill

Automates two LinkedIn workflows via Playwright MCP on an **already logged-in** browser session:

1. **Recruiter Outreach** — Search LinkedIn for recruiters hiring for target roles, then send personalized connection requests with a note (Connect → Add note → Send).
2. **Message Follow-Up** — Scan recent LinkedIn messaging threads, identify conversations without a reply, and send a follow-up message.

---

## IMPORTANT: Before You Start

> **LinkedIn ToS Warning**: Automated actions on LinkedIn can trigger account restrictions. This skill
> is designed for low-volume, human-supervised outreach — NOT mass-blast automation. Always:
> - Add random delays (3-8 seconds) between actions
> - Review personalized messages before sending
> - Stop immediately if LinkedIn shows a CAPTCHA or restriction warning

> **Prerequisite**: LinkedIn must already be logged in within the Playwright MCP browser session.
> Confirm this by navigating to `https://www.linkedin.com/feed/` and checking that the feed loads
> (not a login page). If not logged in, instruct the user to log in manually first.

**Remembered local defaults:**
- Use the user's existing logged-in browser session for LinkedIn browser work.
- Do not ask for the LinkedIn password or start a fresh browser profile unless the user explicitly changes this.
- Paid InMail is disallowed. If a job-page `Message` opens a paid InMail composer, close it and check free `Connect` / `Add a note` on the recruiter profile, including under `More`.
- For recruiter outreach tied to job applications, do not mark outreach skipped until free `Message`, free `Connect`, and `Add a note` paths have been checked.
- Do not navigate directly to LinkedIn invite/preload URLs such as `/preload/custom-invite/` as a shortcut. Use the normal visible LinkedIn UI flow from search results or profile pages (`Connect` / `More` / `Add a note`) and keyword/filter inputs; direct invite URLs can trigger LinkedIn security checks.
- For recruiter search runs, always click the visible `Actively hiring` filter before collecting profiles, then restrict collection to recruiters located in the [user's preferred region, default: United States]. Do not collect or contact inactive recruiters or recruiters outside the target region.

---

## Flow 1: Recruiter Outreach

Read `references/recruiter-outreach-flow.md` for the full step-by-step Playwright MCP implementation.

### High-Level Steps

1. **Gather user context** — Ask for (or pull from resume/memory):
   - Target role titles (e.g., "Backend Engineer", "Software Engineer")
   - Target companies (optional, empty = all)
   - Geographic preference (e.g., "San Francisco Bay Area", "Remote")
   - If no specific role list is provided, loop through these default hiring-for job titles in order: [customizable based on user's domain, default: Software Engineer, Backend Engineer, Full Stack Engineer, Python Engineer, DevOps Engineer, Cloud Engineer].

2. **Build the search query** — Navigate to LinkedIn People Search with filters:
   - Keywords: `recruiter` or `talent acquisition` or `hiring`
   - Title filter: "Recruiter" OR "Talent Acquisition" OR "Technical Recruiter" OR "Hiring Manager"
   - Location filter: always set to [user's preferred region] for recruiter outreach unless the user explicitly overrides it.
   - Always enable LinkedIn's visible `Actively hiring` filter before collecting recruiter candidates.
   - In the `Actively hiring` filter panel, select `Hiring for job title`, enter the current role from the role-title loop, click the visible matching suggestion, then click `Show results`.
   - Treat `Actively hiring` plus region filter as hard requirements. If either filter is unavailable or cannot be verified from the UI/text, stop and report the blocker instead of collecting inactive or non-target recruiters.
   - Repeat search, visible filter selection, scraping, and sending for each role title until the requested count is reached or no eligible recruiters remain; do not reuse already-contacted or pending profiles.

3. **Scrape recruiter profiles from search results** — For each result on the page:
   - Extract: name, headline, company, profile URL, connection degree
   - Skip if already 1st-degree connection (already connected)
   - Skip if the result is not visible after applying `Actively hiring`.
   - Skip if the location is missing or is outside the target region.
   - Skip if the result appears inactive, generic, sponsored, or unrelated to recruiting/talent acquisition.
   - Stop when reaching the user's max count

4. **For each recruiter, send a connection request with a personalized note:**
   - **If "Connect" is visible** → Click it → Wait for "Add a note" modal → Click "Add a note" → Fill personalized message → Click Send
   - **If "Follow" is visible (Creator Mode)** → Click "More" → Click "Connect" from dropdown → Wait for "Add a note" modal → Click "Add a note" → Fill personalized message → Click Send
   - **If paid InMail opens** → Close it immediately, then check for a free Connect / Add a note path (including under "More").
   - **If the visible "Add a note to your invitation?" popup opens** → Click "Add a note", fill the note, and send. Do not stop just because an iframe/detector looks noisy.
   - **If neither available** → Skip immediately
   - **DO NOT** attempt InMail. **DO NOT** mention applying to a specific role. Always use "Add a note" — never send a bare connection request.
   - **DO NOT** open invite/preload URLs directly. If clicks are intercepted, retry through visible UI controls or search/filter keywords rather than URL-forcing the invitation modal.

5. **Generate a summary report** — Show the user:
   - How many recruiters found
   - How many connection requests sent vs. skipped
   - List of names, companies, and action taken

### Personalized Message Templates

The skill generates personalized connection notes using the user's resume context.

> **SETUP REQUIRED:** Replace the template below with your own professional summary.

#### Connection Note Template (300 char limit)
```
Hi [Name], I'm a [your role, e.g., backend/full-stack engineer] with [X years] at
[Company] building [technologies/achievements]. I'm exploring new opportunities and
would love to connect if you have a moment. Thanks!
```

**Personalization variables** (pulled from user's resume or memory):
- `recruiter_first_name`: First name from search results
- `job_title`: The role title being targeted (e.g., "Software Engineer")
- `top_2_skills`: Top 2 relevant skills (e.g., "Java and Spring Boot")
- `years_exp`: Years of experience (e.g., "3+")
- `company`: Recruiter's company from their profile

---

## Flow 2: Message Follow-Up

Read `references/follow-up-flow.md` for the full step-by-step Playwright MCP implementation.

### High-Level Steps

1. **Navigate to LinkedIn Messaging** — Go to `https://www.linkedin.com/messaging/`

2. **Scan recent conversation threads** — For each visible thread:
   - Extract: contact name, last message preview, timestamp, who sent last message (user or contact)
   - Filter for threads where **the user sent the last message** AND **no reply received** within a configurable window (default: 3+ days ago)

3. **For each stale thread, compose a follow-up** — Generate a contextual follow-up based on:
   - The contact's name and title
   - The last message the user sent (to avoid repeating themselves)
   - Time elapsed since last message

4. **Present follow-ups for user approval** — Show each proposed follow-up message to the user before sending. User can:
   - Approve and send
   - Edit and send
   - Skip

5. **Send approved follow-ups** — Click into each thread and send the message.

6. **Generate summary** — Show how many follow-ups sent, skipped, and pending.

### Follow-Up Message Template
```
Hi {contact_first_name}, just circling back on my earlier message.
I remain very interested and would love to connect when you have
a moment. Happy to work around your schedule. Thanks!
```

For recruiter-specific follow-ups:
```
Hi {contact_first_name}, following up on my note about {role_type}
opportunities at {company}. I'd welcome the chance to chat — please
let me know if there's a good time. Thanks!
```

---

## Playwright MCP Implementation Notes

### Session & Navigation
- Always verify login state before starting any flow
- Use browser navigation to go to URLs
- Use text extraction or DOM evaluation to read page content
- Wait for page loads with appropriate selectors before scraping

### Selectors (LinkedIn's DOM — may change, verify before running)
These are **starting-point selectors**. LinkedIn updates its DOM frequently. Use this fallback
chain in strict order — each level is cheaper on tokens than the next:

**Level 1 — Text-based selectors (try FIRST, most stable across redesigns):**
Use text selectors: `text=Connect`, `text=Add a note`,
`text=Send`, `text=More`. These survive most UI reshuffles since button labels rarely change.

**Level 2 — ARIA label selectors:**
`button[aria-label*="connect" i]`, `button[aria-label*="InMail"]`, etc. Slightly more
fragile than text but still resilient.

**Level 3 — CSS class selectors from the reference files:**
The reference files list class-based selectors (`.entity-result__title-text`, etc.). Try
these next — they break more often but are precise when they work.

**Level 4 — DOM inspection via browser evaluation (lightweight, no tokens wasted on images):**
Run JS to enumerate what's on the page. Examples:
```js
// List all buttons with their text and aria-labels
document.querySelectorAll('button').forEach(b => console.log(b.textContent.trim(), b.getAttribute('aria-label')))
```
```js
// Find all clickable elements near a keyword
document.querySelectorAll('[class*="connect"], [class*="message"], [class*="invite"]')
```
This returns structured text, NOT an image — far cheaper than a screenshot.

**Level 5 — Full page text extraction:**
Grab the full page text to understand what state the page is in (logged out? CAPTCHA?
different layout?). Still text-only, still cheap.

**Level 6 (LAST RESORT ONLY) — Screenshots:**
Only take a screenshot if Levels 1-5 ALL failed and you genuinely cannot determine
the page state from text alone. Screenshots consume massive token budgets. Before
screenshotting, ask yourself: "Can I figure this out from text?" — if yes, don't screenshot.

### Rate Limiting & Safety
- **Mandatory delay**: Wait minimum 3 seconds between actions, randomize 3-8s
- **Session cap**: Stop after the configured max (default 10, hard cap 20 per session)
- **CAPTCHA detection**: After each action, check if the page contains CAPTCHA elements or restriction banners. If detected, **stop immediately** and inform the user.
- **Daily limit tracking**: Keep a running count across sessions if possible. Warn the user if approaching 50 total actions in a day.

### Error Handling
- If a selector is not found, do NOT retry blindly — use text selectors first, then DOM evaluation to enumerate elements as structured text. Do NOT screenshot unless all text-based inspection fails.
- If LinkedIn redirects to a login page mid-session, stop and inform the user
- If a connection request fails (button greyed out, "pending" state), log and skip
- Always wrap each recruiter interaction in a try/catch equivalent — one failure should not abort the entire run

---

## User Interaction Model

This skill should **always** operate in a human-in-the-loop mode:

1. **Before starting**: Confirm target roles, companies, location, and max count with the user
2. **Before sending each message** (first run only): Show the personalized message and get approval. After the user approves the template style for 2-3 messages, the skill can batch-send the rest with a summary.
3. **On any error or restriction**: Stop and inform the user immediately
4. **After completion**: Show the full summary report

---

## Quick Reference: Which Flow to Use

| User says | Flow |
|-----------|------|
| "reach out to recruiters" | Flow 1: Recruiter Outreach |
| "find recruiters hiring for [role] roles" | Flow 1: Recruiter Outreach |
| "send connection requests to recruiters" | Flow 1: Recruiter Outreach |
| "check my LinkedIn messages" | Flow 2: Message Follow-Up |
| "follow up on unanswered messages" | Flow 2: Message Follow-Up |
| "send follow-ups on LinkedIn" | Flow 2: Message Follow-Up |
| "do both outreach and follow-ups" | Run Flow 2 first (quick), then Flow 1 |
