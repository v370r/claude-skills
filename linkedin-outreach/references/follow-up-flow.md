# Message Follow-Up Flow — Playwright MCP Steps

Detailed step-by-step browser automation for scanning LinkedIn messages and sending follow-ups to unanswered conversations.

---

## Step 1: Verify Login & Navigate to Messaging

```
Action: playwright_navigate
URL: https://www.linkedin.com/messaging/

Action: playwright_wait(3000)

# Verify we're on the messaging page (not redirected to login)
Action: playwright_get_text
Selector: .msg-conversations-container__title-row  (or h1 containing "Messaging")

Expected: Page shows messaging interface.
If NOT: STOP → tell user to log in.
```

---

## Step 2: Scrape Conversation Threads

### 2a. Get the list of recent conversations

```
Action: playwright_evaluate
JavaScript: |
  const threads = [];
  const items = document.querySelectorAll('.msg-conversation-listitem');
  
  items.forEach(item => {
    const nameEl = item.querySelector('.msg-conversation-listitem__participant-names');
    const previewEl = item.querySelector('.msg-conversation-card__message-snippet');
    const timeEl = item.querySelector('.msg-conversation-listitem__time-stamp, time');
    const unreadBadge = item.querySelector('.notification-badge');
    
    if (nameEl) {
      threads.push({
        name: nameEl.textContent.trim(),
        preview: previewEl ? previewEl.textContent.trim() : '',
        timestamp: timeEl ? timeEl.textContent.trim() : '',
        hasUnread: !!unreadBadge,
        // We'll use index to click into the thread later
        index: threads.length
      });
    }
  });
  
  JSON.stringify(threads);
```

### 2b. Filter for follow-up candidates

A thread needs follow-up if:
- The **user** sent the last message (not the contact)
- It's been **3+ days** since the last message (configurable)
- There is **no unread badge** (meaning the contact hasn't replied)

To determine who sent the last message, we need to click into each thread.

---

## Step 3: Inspect Each Thread

For each conversation thread from Step 2:

### 3a. Click into the thread

```
Action: playwright_click
Selector: .msg-conversation-listitem:nth-child({index + 1})  
# Or use the name-based selector:
# Selector: [data-control-name="view_message"] that contains the contact's name

Action: playwright_wait(2000)
```

### 3b. Extract conversation details

```
Action: playwright_evaluate
JavaScript: |
  const messages = [];
  const msgElements = document.querySelectorAll('.msg-s-event-listitem');
  
  msgElements.forEach(msg => {
    const senderEl = msg.querySelector('.msg-s-message-group__profile-link, .msg-s-message-group__name');
    const bodyEl = msg.querySelector('.msg-s-event-listitem__body');
    const timeEl = msg.querySelector('.msg-s-message-group__timestamp, time');
    
    if (bodyEl) {
      messages.push({
        sender: senderEl ? senderEl.textContent.trim() : 'unknown',
        body: bodyEl.textContent.trim(),
        timestamp: timeEl ? timeEl.textContent.trim() : ''
      });
    }
  });
  
  // Get the last message
  const lastMsg = messages[messages.length - 1];
  
  // Get contact profile info from the thread header
  const contactName = document.querySelector('.msg-thread__link-to-profile, .msg-entity-lockup__entity-title')?.textContent?.trim() || '';
  const contactTitle = document.querySelector('.msg-entity-lockup__subtitle, .msg-thread__participant-info')?.textContent?.trim() || '';
  
  JSON.stringify({
    contactName,
    contactTitle,
    messageCount: messages.length,
    lastMessage: lastMsg,
    allMessages: messages.slice(-5)  // Last 5 messages for context
  });
```

### 3c. Determine if follow-up is needed

Check these conditions:
1. **Last message sender**: Compare `lastMsg.sender` against the user's name. If the user sent the last message → candidate for follow-up.
2. **Time elapsed**: Parse the timestamp. If it's 3+ days old → needs follow-up.
3. **Content check**: If the last message was itself a follow-up (contains phrases like "following up", "circling back", "checking in"), consider whether a second follow-up is appropriate. Generally, skip if the user already followed up once — flag for user decision.

### 3d. Categorize the thread

```javascript
// Category logic
const category = determineCategory(threadData);
// Categories:
// - 'needs_followup': User sent last message 3+ days ago, no prior follow-up
// - 'already_followed_up': User already sent a follow-up, still no reply
// - 'awaiting_response': User sent last message but less than 3 days ago
// - 'replied': Contact sent last message (no action needed)
// - 'skip': Thread is too old (30+ days), or automated/system message
```

---

## Step 4: Generate Follow-Up Messages

For each thread categorized as `needs_followup`:

### Context-aware message generation

Use the thread context (contact name, title, last messages) to generate a personalized follow-up:

**For recruiter/hiring conversations:**
```
Hi {contact_first_name}, just wanted to follow up on my earlier message about 
{role_type} opportunities. I'm still very interested and would love to connect 
when you have a moment. Happy to work around your schedule!
```

**For general networking conversations:**
```
Hi {contact_first_name}, hope you're doing well! Just circling back on my 
earlier note. Would still love to connect — let me know if there's a good time 
to chat.
```

**For conversations where the user asked a specific question:**
```
Hi {contact_first_name}, just bumping this in case it got buried. 
{brief_reference_to_original_question} — appreciate any thoughts when you 
get a chance!
```

### Contextual cues for message type selection

Scan the contact's title and the conversation history:
- Title contains "recruiter", "talent", "hiring", "HR" → recruiter template
- Conversation mentions "role", "position", "opportunity", "resume" → recruiter template
- Conversation mentions "project", "collaboration", "meet" → networking template
- User asked a question in the last message → question follow-up template

---

## Step 5: Present Follow-Ups for Approval

**Critical: Always show follow-ups to the user before sending.**

Present a summary like:

```
Found {count} conversations needing follow-up:

1. {contact_name} ({contact_title}) — Last message: {days} days ago
   Your last message: "{truncated_last_message}..."
   Proposed follow-up: "{proposed_message}"
   [Send] [Edit] [Skip]

2. {contact_name} ({contact_title}) — Last message: {days} days ago
   ...
```

Wait for user approval on each (or batch approval).

---

## Step 6: Send Approved Follow-Ups

For each approved follow-up:

### 6a. Navigate to the thread (if not already there)

```
Action: playwright_navigate
URL: https://www.linkedin.com/messaging/

Action: playwright_wait(2000)

# Click into the specific thread
Action: playwright_click
Selector: .msg-conversation-listitem:has-text("{contact_name}")
# Fallback: search for the contact
Action: playwright_fill
Selector: input[aria-label="Search messages"]  (or .msg-search-form__search-field)
Value: "{contact_name}"

Action: playwright_wait(2000)

Action: playwright_click
Selector: .msg-conversation-listitem:first-child  (first search result)

Action: playwright_wait(2000)
```

### 6b. Type and send the message

```
# Focus the message input
Action: playwright_click
Selector: .msg-form__contenteditable  (or div[contenteditable="true"] in the messaging pane)

Action: playwright_wait(500)

# Type the follow-up message
Action: playwright_fill
Selector: .msg-form__contenteditable
Value: "{approved_follow_up_message}"

Action: playwright_wait(1000)

# Send
Action: playwright_click
Selector: button.msg-form__send-button  (or button[type="submit"], or text="Send")

Action: playwright_wait(randomBetween(3000, 6000))
```

### 6c. Verify sent

```
Action: playwright_evaluate
JavaScript: |
  // Check if the last message in the thread matches what we just sent
  const lastMsg = document.querySelector('.msg-s-event-listitem:last-child .msg-s-event-listitem__body');
  const sent = lastMsg && lastMsg.textContent.includes('{first_few_words_of_message}');
  JSON.stringify({ sent });
```

---

## Step 7: Generate Summary Report

```
LinkedIn Message Follow-Up — Session Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total threads scanned:      {total_scanned}
Needing follow-up:          {needs_followup_count}
Follow-ups sent:            {sent_count}
Skipped by user:            {skipped_count}
Already followed up:        {already_followedup_count}
Errors:                     {error_count}

Sent follow-ups to:
1. {name} ({title}) — "{message_preview}..."
2. {name} ({title}) — "{message_preview}..."

Skipped (already followed up once):
1. {name} — last follow-up {days} days ago
```

---

## Edge Cases & Error Handling

### Thread is a group conversation
Skip group conversations entirely — follow-ups should be 1:1 only.
Detection: If `contactName` contains commas or "and", or if there are multiple participants, skip.

### Contact has deactivated their account
The thread will show but clicking may lead to an error. Check for:
```
Action: playwright_evaluate
JavaScript: |
  const isDeactivated = document.body.textContent.includes('LinkedIn Member') || 
                        document.body.textContent.includes('deactivated');
  JSON.stringify({ isDeactivated });
```

### Message input is disabled
Some threads may have messaging disabled (e.g., contact turned off messaging). Check:
```
Action: playwright_evaluate
JavaScript: |
  const inputDisabled = !document.querySelector('.msg-form__contenteditable') || 
                        document.querySelector('.msg-form__contenteditable[aria-disabled="true"]');
  JSON.stringify({ inputDisabled });
```

### LinkedIn messaging redesign
If selectors fail broadly, it likely means LinkedIn updated their messaging UI. Follow the
fallback chain defined in SKILL.md (text selectors → ARIA selectors → DOM enumeration via
`playwright_evaluate` → `playwright_get_text` → ask user → screenshot as ABSOLUTE LAST RESORT).
Always exhaust text-based inspection before even considering a screenshot — screenshots consume
massive token budgets and rarely tell you anything that `playwright_evaluate` listing all buttons
and inputs wouldn't reveal more efficiently.

---

## Rate Limiting for Follow-Ups

- Maximum 10-15 follow-ups per session
- Random delay of 5-10 seconds between each send
- If LinkedIn shows any warning about messaging limits → STOP immediately
- Space sessions at least 1 hour apart
