# LinkedIn Auto-Apply & Recruiter Outreach — Claude Code Skills

**Are you a developer spending hours scrolling LinkedIn, tailoring resumes, and manually filling out job applications?**

These two Claude Code skills automate the most time-consuming parts of your job search:

- **Auto-apply** to LinkedIn Easy Apply jobs that match your background — no more copy-pasting the same answers 50 times
- **Recruiter outreach** via free connection requests — reach hiring managers directly without spending InMail credits
- **Message follow-ups** — automatically follow up on unanswered recruiter messages

Built for developers who want to focus on interview prep, not form-filling.

> **Important:** Claude Code looks for files named `skill.md` and `SKILL.md`. This repo ships `TEMPLATE.md` files with `[placeholders]`. You copy each template to its live name, fill in your info, then symlink. Your filled-in `skill.md` / `SKILL.md` are `.gitignore`d so personal info never gets committed.

---

## What It Does

### Skill 1: `/linkedin-apply`

Searches LinkedIn Jobs for Easy Apply roles matching your target titles, applies to jobs that meet your fit criteria, and optionally reaches out to listed recruiters.

**Features:**
- Searches multiple job titles in sequence (e.g., Backend Engineer, Full Stack Engineer, Software Engineer)
- Filters for: past 24 hours, Easy Apply only, Mid-Senior level, remote/US-based
- Auto-decides whether to apply based on match score (40%+) or role signal matching (2+ relevant skills)
- Fills application forms from your profile info, resume, and defaults
- Writes tailored "Why are you interested?" responses matched to each job posting
- Reaches out to recruiters listed on the job via free Connect + Message paths
- Never spends InMail credits — falls back to free connection requests automatically

### Skill 2: `/linkedin-outreach`

Searches for recruiters actively hiring for your target roles, sends personalized connection requests, and follows up on unanswered messages.

**Features:**
- Searches LinkedIn People for recruiters with "Actively hiring" status
- Filters by location and role-specific hiring
- Sends personalized connection notes (under 300 characters)
- Follows up on stale messaging threads where you sent the last message
- Human-in-the-loop: approves messages before sending
- Rate-limited with delays to avoid triggering LinkedIn restrictions

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Claude Code** | The CLI tool ([install guide](https://docs.anthropic.com/en/docs/claude-code/overview)) |
| **LinkedIn account** | Active session in your browser |
| **Playwright MCP** (outreach only) | Required for `/linkedin-outreach` browser automation. The `/linkedin-apply` skill uses the connected browser directly. |
| **Your profile info** | You must customize the skill templates with your own education, experience, skills, and preferences |

---

## Quick Start

### 1. Clone this repo

```bash
git clone https://github.com/YOUR_USERNAME/claude-linkedin-skills.git
cd claude-linkedin-skills
```

### 2. Customize the skill templates

Each skill has a `TEMPLATE.md` file with `[placeholders]` for your personal info. Replace them:

**For `linkedin-apply`:**
```bash
cp claude-skills/linkedin-apply/TEMPLATE.md claude-skills/linkedin-apply/skill.md
```

Open `claude-skills/linkedin-apply/skill.md` and replace:
- `[Degree]`, `[University]`, `[GPA]`, `[graduation date]` — your education
- `[X years] [Job Title] at [Company]` — your work experience
- `[key technologies and accomplishments]` — your tech stack
- `[Target roles]` — job titles you want to apply to
- `[Preferred roles]` — remote/hybrid/on-site, locations
- Work authorization defaults (F1-OPT, US citizen, etc.)

**For `linkedin-outreach`:**
```bash
cp claude-skills/linkedin-outreach/TEMPLATE.md claude-skills/linkedin-outreach/SKILL.md
```

Open `claude-skills/linkedin-outreach/SKILL.md` and replace:
- Connection note template with your own professional summary
- Target role titles for recruiter search
- Preferred geographic region

### 3. Install into Claude Code

**Windows (PowerShell):**
```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\linkedin-apply" -Target "<full-path-to-repo>\claude-skills\linkedin-apply"
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\linkedin-outreach" -Target "<full-path-to-repo>\claude-skills\linkedin-outreach"
```

**macOS / Linux:**
```bash
ln -s "$(pwd)/claude-skills/linkedin-apply" ~/.claude/skills/linkedin-apply
ln -s "$(pwd)/claude-skills/linkedin-outreach" ~/.claude/skills/linkedin-outreach
```

> **Symlinks vs. copy:** Symlinks let you `git pull` updates that apply immediately. If you copy instead, you'll need to recopy after each update.

### 4. Restart Claude Code

The `/linkedin-apply` and `/linkedin-outreach` commands will now be available.

---

## Usage

### Apply to Jobs

```
/linkedin-apply
```

This will:
1. Search LinkedIn Jobs for your configured target roles
2. Filter for Easy Apply, past 24 hours, appropriate experience level
3. Auto-apply to matching jobs (default: max 10 per run)
4. Optionally reach out to listed recruiters

### Recruiters Outreach

```
/linkedin-outreach
```

This will:
1. Search for recruiters actively hiring for your target roles
2. Send personalized connection requests (with approval)
3. Follow up on unanswered messages

### Customizing Your Run

You can override defaults in your prompt:

```
/linkedin-apply — limit to 5 applications, focus on Python roles only
/linkedin-outreach — target only FAANG companies, max 15 connections
```

---

## Syncing Across Machines

Since skills are symlinked to this repo, updating is just a `git pull`:

```bash
cd /path/to/claude-linkedin-skills
git pull
```

On a new machine, repeat steps 1-4 above. Your personalized `skill.md` / `SKILL.md` files will need to be created from the templates on each machine.

> **Tip:** If you use the same profile across machines, commit your filled-in `skill.md` / `SKILL.md` to the repo (add them to `.gitignore` for sensitive info, or use a private repo).

---

## Safety & Best Practices

### LinkedIn Terms of Service
- This tool operates through your logged-in browser session — LinkedIn's normal UI
- Actions are rate-limited (3-8 second delays between actions)
- **Never** use this for mass-blast automation — keep volumes reasonable (10-20 per session)
- **Stop immediately** if LinkedIn shows a CAPTCHA or restriction warning
- These are guidelines to protect your account — not legal advice

### What This Skill Does NOT Do
- Bypass CAPTCHAs, paywalls, or account security
- Spend InMail credits
- Fabricate answers on application forms
- Apply to jobs that clearly don't match your background
- Post to your profile or send messages on your behalf without approval

### Hard Limits
| Setting | Default | Notes |
|---|---|---|
| Max applications per run | 10 | Overrideable in prompt |
| Max outreach per session | 10 | Hard cap 20 |
| Daily action limit | 50 | Warning threshold |
| Delay between actions | 3-8s | Randomized |

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Skill not found after install | Restart Claude Code. Verify symlink: `ls ~/.claude/skills/` |
| LinkedIn filter panel error | Click Search button, or navigate to linkedin.com/jobs/ first, then retry |
| "No matching jobs" but jobs visible | Filters may not have applied — check the URL for `f_AL=true` (Easy Apply) |
| CAPTCHA appeared | Stop immediately. Complete the CAPTCHA manually. Wait before resuming. |
| Login required mid-session | Log in manually in your browser, then restart the skill |
| Message send fails | LinkedIn may have changed their DOM. Use `browser_evaluate` to inspect the page |
| Connection note too long (300 char) | Shorten the template — the skill auto-truncates if needed |
| Skill uses wrong Chrome profile | Check your browser profile setting in the skill's Operating Rules |

---

## Customization Tips

### Change Target Roles
Edit the search query list in `skill.md` to match your desired job titles:

```markdown
1. `Data Engineer`
2. `Machine Learning Engineer`
3. `Platform Engineer`
```

### Adjust Match Criteria
The default is 40% fit or 2+ matching role signals. You can raise/lower these in the Match section.

### Add Your Own Note Template
Replace the recruiter outreach note template with your own voice and key highlights.

### Configure Work Authorization
If you're on F1-OPT, H1B, or any specific status, update the defaults in the Apply section.

---

## Project Structure

```
claude-skills/
├── README.md                                    # This file
├── linkedin-apply/
│   ├── skill.md                                 # [YOUR FILLED-IN VERSION]
│   └── TEMPLATE.md                              # Template with [placeholders]
└── linkedin-outreach/
    ├── SKILL.md                                 # [YOUR FILLED-IN VERSION]
    ├── TEMPLATE.md                              # Template with [placeholders]
    └── references/
        ├── follow-up-flow.md                    # Playwright implementation details
        └── recruiter-outreach-flow.md           # Playwright implementation details
```

---

## Credits

Built by [Vijay Kumar PolojU](https://www.linkedin.com/in/vijaykumarpolaju) — MS Computer Science, CU Boulder | Backend/Full-Stack Engineer

Feel free to fork, customize, and adapt for your own job search.
