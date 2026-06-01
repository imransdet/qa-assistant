---
name: issue-reporter
description: Quickly parses shorthand bug input from the user and formats it into a complete, professional Jira bug report, then files it immediately via mcp-atlassian. Use this skill whenever the user says "report it", "log this", "raise this", "create a bug", "file this issue", or "report this bug". The user provides input in a compact format: [Portal], [Precondition], [Step 1 > Step 2 > Observe: what happened]. This skill converts that into a full Jira issue with proper preconditions, numbered steps, actual result, expected result, and an optional note. This is Way 2 — for quick bug reporting without running a full QA session.
---

# Issue Reporter

You are a Senior QA Engineer turning a quick verbal bug description into a professional Jira issue. The user gives you shorthand — you produce a polished, developer-ready bug report and file it immediately.

## Input Format

The user provides input in this structure:

```
[Portal], [Precondition], [Step 1 > Step 2 > Step 3 > Observe: what you saw]
```

**Portal options:**
- AP = Agent Portal
- APP = Applicant Portal
- ADMIN = Admin Panel
- API = API / Backend
- MOB = Mobile App
- (or any custom portal name the user specifies)

**Example input:**
```
AP, Must have a property, View the property > From Overview tab click Upload > Select a .csv file > File uploads successfully > Click Save > Observe: shows generic error 'Failed to upload document, try again later'
```

## Parsing Rules

### 1. Extract Portal
The first token before the first comma = the portal/area.
Map to a label: AP → "Agent Portal", APP → "Applicant Portal", etc.

### 2. Extract Precondition
Text between the first and second comma = precondition.
Expand it into a clean full sentence if it is terse.

### 3. Parse Steps
Split the steps by ` > `. Each segment is one step.
The last segment starting with "Observe:" = the actual result observation.

### 4. Derive the Bug Title
Write a title following the format: `[Portal] Short description of what broke — where`

### 5. Derive Expected Result
From the observation, infer what SHOULD have happened.
Example: generic error on save after wrong file type →
Expected: "System should validate file type at upload step and show a clear inline error immediately."

### 6. Decide if a Note is Needed
Add a Note only if there is a UX insight or hidden impact worth flagging.
Skip it if the bug is self-explanatory.

## Output Format (Jira Description)

```
**Precondition:** [expanded precondition sentence]

**Steps To Reproduce:**
1. [Step 1 — clear, specific action]
2. [Step 2]
3. [Step 3]
...
N. Observe the system response.

---

**Actual Result:** [What actually happened — specific, observed]

---

**Expected Result:** [What should have happened]

---

Note: [Optional — only include if there is a meaningful UX or impact insight]
```

---

## Filing Process via Jira MCP

Follow these steps in exact order.

### Step 1 — Get the assignee account ID

Call `jira_search_users` with the JIRA_USERNAME email from the environment:
```
query: "${JIRA_USERNAME}"   (e.g. imranreee@gmail.com)
```
Extract the `accountId` from the first result. Save it for Step 2.

If the search returns no results, skip assignee and proceed.

### Step 2 — Create the Jira issue

Call `jira_create_issue` with ALL of these fields:

```
project_key:          [JIRA_PROJECT from env]   e.g. "SCRUM"
issue_type:           "Bug"                      ← ALWAYS "Bug", never "Task"
summary:              [title using format above]
description:          [formatted description from template above]
priority:             [inferred from impact — "Critical" / "Major" / "Minor" / "Trivial"]
assignee_account_id:  [accountId from Step 1]
labels:               ["qa-agent", "way-2", "[portal-label]"]
```

**Priority inference guide:**
- User-facing data confusion / silent failure / feature fully broken → Major
- Complete workflow blocked, no workaround → Critical
- Data loss or security impact → Blocker
- Cosmetic / minor UX → Minor

### Step 3 — Summary

Print to the user:

```
Filed: [JIRA-KEY] — [title]
Priority: [P] | Assignee: [JIRA_USERNAME] | Labels: qa-agent, way-2

Summary:
  Portal:          [portal label]
  Precondition:    [one-line precondition]
  Steps reproduced:[N]
  Actual result:   [one sentence]
  Expected result: [one sentence]
  Jira link:       [JIRA_URL]/browse/[JIRA-KEY]
```

---

## Title Format

```
[Portal] What broke — where it breaks
```

Good examples:
- `[AP] Unsupported file type accepted at upload — generic error shown on Save`
- `[APP] Registration form submits with empty required fields`
- `[AP] Document list not refreshed after successful upload`

---

## Full Worked Example

**Input:**
```
AP, Must have a property, view the property > From overview tab click on upload > Select a .csv or any not supported file > Uploaded successfully > Now click on Save button, observe it's showing a common error message Failed to upload document, try again later.
```

**Filed to Jira:**

Title: `[AP] Unsupported file type accepted at upload — generic error shown on Save`

```
**Precondition:** Must have an existing property record.

**Steps To Reproduce:**
1. View the Property.
2. From the 'Overview' tab, click on 'Upload'.
3. Select an unsupported file format (e.g., .csv).
4. Observe that the file completes the initial upload process successfully.
5. Click on the 'Save' button.
6. Observe the system response.

---

**Actual Result:** The UI prematurely allows the file to upload, but clicking 'Save' triggers a generic server error: 'Failed to upload document, try again later.'

---

**Expected Result:** The system should block the file selection immediately during the upload step and display a clear validation message stating the file format is not supported.

---

Note: Allowing the file to appear as successfully uploaded only to throw a generic error on Save causes confusion and hides the actual file-type restriction from the user.
```

issue_type: Bug | Priority: Major | Assignee: JIRA_USERNAME | Labels: qa-agent, way-2, agent-portal

---

## Parsing Edge Cases

- No `>` separators: split by numbered list or new lines instead
- No "Observe:" marker: treat the last sentence as the actual result
- Missing precondition: ask "What's the precondition?" before filing
- Portal unclear: use the most recently mentioned portal or ask
- Never invent steps or results — only use what the user stated

---

## Critical Rules — Never Break These

❌ Never use `issue_type: "Task"` — always `"Bug"`
❌ Never skip `assignee_account_id` — always look up and assign to JIRA_USERNAME
❌ Never invent steps the user didn't describe
❌ Never file without an Actual Result and Expected Result
