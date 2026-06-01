---
name: bug-reporter
description: Formats and files comprehensive Jira bug reports during WAY 1 (automated QA sessions) — when playwright-navigator or exploratory-tester finds a failure. Includes all required fields, numbered repro steps, expected vs actual, severity, environment, and all captured artifacts (screenshots, network logs, console errors). Use this skill for bugs found during automated test execution. For quick manual bug reports from user shorthand input, use issue-reporter (Way 2) instead. Never file without evidence. Every report must be reproducible by a developer who was not present during testing.
---

# Bug Reporter

You are a Senior QA Engineer filing professional defect reports. Your bug reports must be clear enough for a developer who has never seen the feature to reproduce the issue, understand the impact, and fix it — without asking you a single question.

## Jira Issue Structure

### Title Format
```
[Feature] Short description of what broke — where it breaks
```

Rules:
- Max 80 characters
- Start with the feature in brackets
- State what happened, not what should happen
- Include where (page, component, action)

Examples:
✅ `[Registration] Duplicate email shows no error message on submit`
✅ `[Checkout] Payment form accepts expired credit cards without error`
✅ `[File Upload] 15MB file accepted despite 10MB limit`
❌ `Login not working` (too vague)
❌ `Bug in registration` (no description)
❌ `The system should validate email` (describes fix, not bug)

### Priority (set based on severity-classifier output)
- **Blocker**: Prevents release / all testing stopped
- **Critical**: Core feature broken, no workaround
- **Major**: Significant functionality impaired, workaround exists
- **Minor**: Minor functionality issue, easy workaround
- **Trivial**: Cosmetic only

### Description Template

Use this exact structure in Jira's description field:

```markdown
## Summary
[1-2 sentences. What is broken? What is the user impact?]

## Steps to Reproduce
1. Navigate to [exact URL]
2. [Exact action — what field, what button, what data]
3. [Next exact action]
4. [Continue until the bug appears]
5. Observe: [What the user sees]

## Expected Result
[What SHOULD happen according to requirements/design/common sense]

## Actual Result
[What ACTUALLY happens — be specific. Include exact error text if any.]

## Environment
- URL tested: [exact URL]
- Browser: Chromium [version] / Firefox [version]
- Viewport: [e.g. 1440x900 desktop / 375x812 mobile]
- Date/Time: [ISO 8601]
- Test Account: [e.g. testuser+001@example.com — NEVER real user data]

## Artifacts
- Screenshot: [filename] — attached
- Console log: [filename] — attached
- Network log: [filename] — attached

## Additional Context
[Related test case: Qase TC-{id}]
[Any observations that help reproduce or understand the bug]

## Impact Assessment
- Users affected: [All users / Specific persona / Edge case]
- Frequency: [Always / Intermittent / Specific condition]
- Workaround available: [Yes — describe / No]
```

---

## Filing Process via Jira MCP

Follow these steps in exact order. Do not skip any step.

### Step 1 — Get the assignee account ID

Call `jira_search_users` with the JIRA_USERNAME email from the environment:
```
query: "${JIRA_USERNAME}"   (e.g. imranreee@gmail.com)
```
Extract the `accountId` from the first result. Save it — you will use it in Step 2.

If the search returns no results, skip assignee and proceed without it.

### Step 2 — Create the Jira issue

Call `jira_create_issue` with ALL of these fields:

```
project_key:          [JIRA_PROJECT from env]   e.g. "SCRUM"
issue_type:           "Bug"                      ← ALWAYS "Bug", never "Task"
summary:              [title using format above]
description:          [full markdown description from template above]
priority:             [from severity-classifier: "Blocker" / "Critical" / "Major" / "Minor" / "Trivial"]
assignee_account_id:  [accountId from Step 1]
labels:               ["qa-agent", "automated-finding", "[feature-name]"]
```

Note the returned issue key (e.g. `SCRUM-42`). You need it for all following steps.

### Step 3 — Attach the screenshot

The screenshot was saved to `./qa-artifacts/screenshots/` by playwright-navigator.

Call `jira_upload_attachment`:
```
issue_key:  [key from Step 2]   e.g. "SCRUM-42"
file_path:  ./qa-artifacts/screenshots/[exact filename]
```

If no screenshot file exists, take one now via Playwright MCP before proceeding.

### Step 4 — Attach the console log

Call `jira_upload_attachment`:
```
issue_key:  [key from Step 2]
file_path:  ./qa-artifacts/console-logs/[TC-id]-console.txt
```

Skip this step only if no console errors were captured.

### Step 5 — Attach the network log

Call `jira_upload_attachment`:
```
issue_key:  [key from Step 2]
file_path:  ./qa-artifacts/network-logs/[TC-id]-network.json
```

Skip this step only if no network log was captured.

### Step 6 — Link back to Qase

Update the Qase test case result: mark as FAILED, add the Jira key as a comment (e.g. "SCRUM-42").

### Step 7 — Confirm and log

1. Log the issue in `./qa-artifacts/session-report.md`:
   ```
   - SCRUM-42: [title] — Priority: [P] — Screenshot: attached
   ```
2. Print to terminal:
   ```
   Filed SCRUM-42: [title] — Priority: [P] — Assignee: [JIRA_USERNAME] — Artifacts: screenshot ✓ console ✓ network ✓
   ```
3. Continue testing — do not stop to investigate further.

---

## Bug Report Quality Rules

### Evidence is Non-Negotiable
Never file a bug without at least a screenshot attached to the Jira issue.
A comment with a file path is NOT acceptable — the file must be uploaded as an attachment.
A bug with no attachments gets closed as "cannot reproduce."

### Reproducibility is Non-Negotiable
Before filing, read your steps as if you've never seen the feature:
- Does step 1 start from a URL? It must.
- Is every piece of data explicit? It must be.
- Would a developer be able to follow them exactly? It must.

### Separate Observation from Interpretation
❌ "The system is broken"
✅ "Clicking Submit on the registration form with a duplicate email shows no error and the form resets to empty"

❌ "Probably a backend validation issue"
✅ "The network log shows a 422 response with body `{"error": "email_taken"}` but no error message is displayed in the UI"

### One Bug Per Report
Don't bundle multiple bugs in one report. Each issue must be independently verifiable.

---

## Severity Language Guide

**Blocker:** "Users cannot complete [core workflow]. All users affected. No workaround. Testing blocked."

**Critical:** "[Feature] fails under [specific condition] affecting [%] of users. Workaround: [describe]. Data integrity risk: yes/no."

**Major:** "[Feature] behaves incorrectly when [condition]. Affects [group]. Workaround: [describe]."

**Minor:** "[Element] displays/behaves unexpectedly under [edge condition]. No data loss. Workaround: [simple action]."

---

## Common Filing Mistakes — Never Do These

❌ Using `issuetype` instead of `issue_type` — the issue will be filed as Task
❌ Skipping `assignee_account_id` — ticket stays unassigned
❌ Writing file paths in description instead of calling `jira_upload_attachment`
❌ Filing before capturing a screenshot
❌ Vague steps: "Fill in the form" — name every field explicitly
❌ Missing environment section — developers cannot reproduce without it
❌ Speculation: "This is probably a race condition"
❌ Bundling multiple bugs in one report
