# Recruiter Outreach Flow — Playwright MCP Steps

Detailed step-by-step browser automation instructions for finding and messaging recruiters on LinkedIn.

---

## Step 1: Verify Login State

```
Action: playwright_navigate
URL: https://www.linkedin.com/feed/

Action: playwright_get_text
Selector: .feed-identity-module__actor-meta  (or fallback: [data-control-name="identity_welcome_message"])

Expected: Page contains the user's name or a feed. 
If NOT: Page shows login form → STOP. Tell user to log in manually via the Playwright browser.
```

---

## Step 2: Build & Execute Search

### Construct the search URL

LinkedIn People Search URL pattern:
```
https://www.linkedin.com/search/results/people/?keywords={query}&origin=GLOBAL_SEARCH_HEADER&titleFreeText={title_filter}&geoUrn={geo_urn}
```

**Recommended search approach** (more reliable than URL params):

```
Action: playwright_navigate
URL: https://www.linkedin.com/search/results/people/?keywords=recruiter%20{target_role}&origin=GLOBAL_SEARCH_HEADER&geoUrn=%5B%22103644278%22%5D

Wait: playwright_wait(3000)
```

### Role-title loop

When the user does not provide a specific list, repeat the search/filter/send workflow for these `Hiring for job title` values in order:

1. Software Engineer
2. Java Engineer
3. Backend Engineer
4. Full Stack Engineer
5. Python Engineer
6. DevOps Engineer
7. Cloud Engineer

For each role title, keep a `seenProfileUrls` set so the same recruiter is not contacted twice across role searches. Continue to the next title when the current title has no more eligible recruiters, or stop when the requested send count is reached.

Then apply filters via the UI if needed:
```
Action: playwright_click
Selector: button[aria-label="All filters"]

Action: playwright_wait(2000)

# Set Title filter
Action: playwright_fill
Selector: input[aria-label="Add a title"]  (or similar — inspect first)
Value: "Recruiter" OR "Technical Recruiter" OR "Talent Acquisition"

# Set Location filter
Action: playwright_fill
Selector: input[aria-label="Add a location"]
Value: "United States"

# Required: enable LinkedIn's Actively hiring filter before collecting any candidates.
# Use the visible filter chip/control. Do not collect results until the UI/text confirms
# the filter is active.
Action: playwright_click
Selector: text="Actively hiring"  (or the visible Actively hiring filter control)

# Required: select the current role title inside the Actively hiring filter panel.
Action: playwright_click
Selector: text="Hiring for job title"

Action: playwright_fill
Selector: input[placeholder="Hiring for job title"]  (or the visible job-title input)
Value: "{current_role_title}"

Action: playwright_click
Selector: visible suggestion matching "{current_role_title}"

# Apply filters
Action: playwright_click
Selector: text="Show results"

Action: playwright_wait(3000)
```

**Hard recruiter-search requirements:**
- The visible `Actively hiring` filter must be selected before scraping.
- The location must be United States (`geoUrn=103644278`) unless the user explicitly overrides it in the same request.
- The visible `Hiring for job title` value must be selected for the current role title before scraping.
- Do not rely on URL parameters alone. Confirm the UI/text shows `People`, `United States`, `Actively hiring`, and the current role title before collecting candidates.
- If the UI does not expose or confirm `Actively hiring`, stop and report that blocker. Do not collect inactive recruiters as a fallback.
- Do not collect or contact recruiters whose result location is missing or outside the United States.

**Alternative — direct URL with encoded filters** (fragile but faster):
```
https://www.linkedin.com/search/results/people/?keywords=technical%20recruiter%20{company}&titleFreeText=recruiter&geoUrn=%5B%22{geo_id}%22%5D
```

Common geo IDs (use web search if needed for others):
- San Francisco Bay Area: `90000084`
- United States: `103644278`
- New York City: `90000070`
- Remote/Anywhere: omit the geoUrn param

---

## Step 3: Scrape Search Results

```
Action: playwright_evaluate
JavaScript: |
  // Extract recruiter cards from search results
  const results = [];
  const cards = document.querySelectorAll('.reusable-search__result-container');
  
  cards.forEach(card => {
    const nameEl = card.querySelector('.entity-result__title-text a span[aria-hidden="true"]');
    const headlineEl = card.querySelector('.entity-result__primary-subtitle');
    const locationEl = card.querySelector('.entity-result__secondary-subtitle');
    const profileLink = card.querySelector('.entity-result__title-text a');
    const connectionDegree = card.querySelector('.dist-value');
    
    if (nameEl) {
      results.push({
        name: nameEl.textContent.trim(),
        headline: headlineEl ? headlineEl.textContent.trim() : '',
        location: locationEl ? locationEl.textContent.trim() : '',
        profileUrl: profileLink ? profileLink.href : '',
        degree: connectionDegree ? connectionDegree.textContent.trim() : 'unknown'
      });
    }
  });
  
  JSON.stringify(results);
```

### Filter results:
- Remove entries where `degree === "1st"` (already connected)
- Remove entries with empty `profileUrl`
- Remove entries whose `location` is empty or not in the United States. Accept U.S. city/state text, `United States`, and common U.S. metro names only when the page was already filtered to United States.
- Remove entries whose headline/title does not indicate an active recruiter, talent acquisition, sourcer, HR, staffing, or hiring role.
- Remove entries if the current results page is not confirmed to be filtered by `Actively hiring`.
- Truncate to the user's configured max (default 10)

### Pagination (if needed to fill max count):
```
Action: playwright_click
Selector: button[aria-label="Next"]  (or .artdeco-pagination__button--next)

Action: playwright_wait(3000)

# Re-run the scraping script above
```

**Important**: LinkedIn usually shows ~10 results per page. If the user wants 15, you'll need page 2. Cap at 2-3 pages max to avoid detection.

---

## Step 4: Visit Each Profile & Send Outreach

For each recruiter in the filtered list:

### 4a. Navigate to profile
```
Action: playwright_navigate
URL: {recruiter_profile_url}

Action: playwright_wait(randomBetween(3000, 6000))
```

### 4b. Extract profile details for personalization
```
Action: playwright_evaluate
JavaScript: |
  const name = document.querySelector('.text-heading-xlarge')?.textContent?.trim() || '';
  const headline = document.querySelector('.text-body-medium')?.textContent?.trim() || '';
  const company = document.querySelector('.inline-show-more-text')?.textContent?.trim() || '';
  
  // Check available action buttons
  const hasConnect = !!document.querySelector('button[aria-label*="connect" i], button[aria-label*="Connect"]');
  const isPending = !!document.querySelector('button[aria-label*="Pending"]');
  const hasMore = !!document.querySelector('button[aria-label="More actions"]');
  
  JSON.stringify({ name, headline, company, hasConnect, isPending, hasMore });
```

### 4c. Determine action — ALWAYS Connect → Add note → Send

**The ONLY action to take on every recruiter profile is:**
Click Connect → Add a note → Fill personalized message → Click Send

**DO NOT** mention applying to any specific role in the note.
**DO NOT** attempt InMail or any other action.
**DO NOT** skip profiles unless already connected or pending.

**Three button states on a profile — handle each:**

1. **"Connect" button visible** → Click it → Wait for "Add a note" modal → Click "Add a note" → Fill note → Click Send
2. **"Follow" visible (Creator Mode)** → Click "More" → Click "Connect" from dropdown → Wait for "Add a note" modal → Click "Add a note" → Fill note → Click Send
3. **Neither Connect nor Follow visible** → Skip immediately (log as `skipped_no_option`)

**Skip only if:**
- Already 1st-degree connection
- Request already pending (`isPending`)
- No Connect button available after checking "More" dropdown

### 4d. Send Connection Request with Note (the ONLY outreach action)

**Scenario 1 — "Connect" button visible:**

```
# Click Connect button
Action: playwright_click
Selector: button[aria-label*="connect" i]  (or the primary Connect button)

Action: playwright_wait(2000)

# LinkedIn shows "How do you know {name}?" modal — click "Add a note"
Action: playwright_click
Selector: button[aria-label="Add a note"]  (or text="Add a note")

Action: playwright_wait(1000)

# Fill the note (300 character limit!)
Action: playwright_fill
Selector: textarea#custom-message  (or textarea[name="message"])
Value: "{generated_connection_note}"  # MUST be under 300 characters

# Send the request
Action: playwright_click
Selector: button[aria-label="Send invitation"]  (or text="Send")  (or button.ml1)

Action: playwright_wait(randomBetween(3000, 8000))
```

**Scenario 2 — "Follow" visible (Creator Mode), Connect hidden under "More":**

```
# Click "More" dropdown
Action: playwright_click
Selector: button[aria-label="More actions"]  (or .pvs-profile-actions__overflow-toggle)

Action: playwright_wait(1000)

# Click Connect from the dropdown
Action: playwright_click
Selector: [aria-label*="connect" i]  (inside the dropdown menu)

Action: playwright_wait(2000)

# Then continue: Add a note → fill → send (same as Scenario 1 above)
```

**Scenario 3 — Neither Connect nor Follow visible:**
Log as `skipped_no_option` and move to next profile immediately.

---

## Step 5: Track Results & Generate Report

Maintain a results array throughout the run:

```javascript
const results = [];
// After each recruiter interaction:
results.push({
  name: recruiterName,
  company: recruiterCompany,
  profileUrl: profileUrl,
  action: 'connection_sent' | 'skipped_pending' | 'skipped_no_option' | 'error',
  message: messageSent,  // the actual personalized message
  error: errorMessage || null
});
```

### Summary Report Format

Present to user as a clean table:

```
LinkedIn Recruiter Outreach — Session Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total recruiters found:      {count}
Connection requests sent:    {connect_count}
Skipped (already pending):   {pending_count}
Skipped (no option):         {no_option_count}
Errors:                      {error_count}

Details:
1. {name} @ {company} — {action}
2. {name} @ {company} — {action}
...
```

---

## Selector Fallback Strategy

LinkedIn's DOM changes frequently. If primary selectors fail, follow this chain in strict
order — each level is cheaper on tokens than the next:

1. **Text-based selectors (cheapest, most stable)** — `text=Connect`, `text=Send InMail`, `text=Add a note`, `text=Send`, `text=More actions`. Button labels rarely change across redesigns.
2. **ARIA label selectors** — `button[aria-label*="connect" i]`, `button[aria-label*="InMail"]`. Slightly less stable but still resilient.
3. **DOM enumeration via `playwright_evaluate`** — Run lightweight JS to list what's on the page:
   ```js
   // List all buttons with their labels
   JSON.stringify(
     Array.from(document.querySelectorAll('button'))
       .map(b => ({ text: b.textContent.trim().slice(0, 50), aria: b.getAttribute('aria-label') }))
       .filter(b => b.text || b.aria)
   );
   ```
   This returns structured text — no image tokens consumed. Use the output to find the correct selector.
4. **`playwright_get_text` on the page** — Grab the visible page text to understand what state you're in (wrong page? logged out? CAPTCHA?). Still text-only.
5. **Ask the user** — Describe what selectors you tried and what text you found on the page. The user can manually inspect and tell you the right selector.
6. **`playwright_screenshot` (ABSOLUTE LAST RESORT)** — Only if Levels 1-5 all failed AND you cannot determine the page state from text. Screenshots consume massive token budgets and should be avoided unless truly stuck with zero text-based signal.

---

## CAPTCHA & Restriction Detection

After every action, run this check:

```
Action: playwright_evaluate
JavaScript: |
  const hasCaptcha = !!document.querySelector('#captcha-internal, .captcha-module, iframe[src*="captcha"]');
  const hasRestriction = document.body.textContent.includes('restricted') || 
                         document.body.textContent.includes('unusual activity') ||
                         document.body.textContent.includes('verify');
  JSON.stringify({ hasCaptcha, hasRestriction });
```

If either is true → **STOP ALL AUTOMATION IMMEDIATELY**. Inform the user and suggest:
- Wait 24 hours before trying again
- Reduce the outreach volume
- Complete any CAPTCHA manually
