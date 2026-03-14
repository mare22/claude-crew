---
name: review-task
description: "Review a task in human-review status. Approve it to done, or reject it back to todo with feedback and optional screenshots. Triggers on: review task, review story, human review, approve task, reject task."
user-invocable: true
---

# Review Task

Review tasks that are in `human-review` status. Approve them or send them back to `todo` with feedback.

---

## Step 1: Read the Board

1. Read `tasks/board.json`
2. Find all stories with `"status": "human-review"`
3. If there are NONE, tell the user: "No tasks in human-review. Nothing to review." and STOP.

---

## Step 2: Show Tasks for Review

If there's only ONE task in human-review, show it directly.

If there are MULTIPLE, list them and ask which one to review:

```
Tasks ready for human review:

1. [ID]: [Title]
   Notes: [notes from QA/design review]

2. [ID]: [Title]
   Notes: [notes from QA/design review]

Which task do you want to review? (number or ID)
```

Wait for the user's choice.

---

## Step 3: Show Task Details

Display the full task details:

```
## [ID]: [Title]

**Type:** [type, if present]
**Description:** [description]
**Priority:** [priority]

### Acceptance Criteria
- [criterion 1]
- [criterion 2]
- ...

### Notes (from agents)
[notes field — includes QA results, design review results, etc.]

---

What's your verdict?

1. ✅ Approve — move to "done"
2. ❌ Reject — send back to "todo" with feedback
```

Wait for the user's answer.

---

## Step 4A: Approve

If the user approves:

1. Read `tasks/board.json`
2. Set the story's `"status": "done"`
3. Append to notes: `"Human review: Approved."`
4. Write `board.json`
5. Tell the user: `"[ID]: [Title] → done ✅"`

---

## Step 4B: Reject — Collect Feedback

If the user rejects, check if they already provided feedback in their message (e.g., `/review-task US-003 the button styling is wrong`).

### If NO feedback was provided, ask:

```
What's wrong with this task? You can:

1. Type your feedback
2. Provide screenshot paths (I'll copy them to tasks/)
3. Both

Please describe what needs to be fixed:
```

Wait for the user's response.

### If feedback was provided, proceed directly.

---

## Step 5: Process Rejection

1. **Copy any screenshots** the user referenced into the `tasks/` directory:
   - Copy each file to `tasks/review-[story-id]-[original-filename]`
   - Example: `~/Desktop/bug.png` → `tasks/review-US-003-bug.png`

2. **Read `tasks/board.json`** (fresh read)

3. **Update the story:**
   - Set `"status": "todo"`
   - Set `"priority": 0` (highest priority — gets picked up next)
   - Set `"notes"` to:
     ```
     HUMAN REVIEW REJECTED:
     - [User's feedback]
     - Screenshots: [list of copied screenshot files, if any]

     PREVIOUS NOTES:
     [original notes content]
     ```

4. **Write `board.json`**

5. **Confirm to the user:**
   ```
   [ID]: [Title] → back to todo (priority 0 — will be picked up next)

   Feedback saved:
   - [feedback summary]
   - Screenshots: [list, if any]

   Run /start-crew → Developers to have it fixed.
   ```

---

## Handling Multiple Reviews

After completing a review, check if there are more tasks in `human-review`:

- If YES: Ask "There are [N] more tasks in human-review. Want to review another?"
- If NO: "No more tasks to review."
