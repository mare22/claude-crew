---
name: add-task
description: "Add a new task (feature, improvement, or bug) to tasks/board.json. Accepts a freeform description, generates a structured user story, and inserts it at the chosen priority. Triggers on: add task, new task, add story, add bug, add feature, add improvement."
user-invocable: true
---

# Add Task

Adds a new task to `tasks/board.json` from a freeform description.

---

## Step 1: Read the Board

1. Read `tasks/board.json`
2. If the file doesn't exist, tell the user to run `/build-board` first and STOP.
3. Note the existing story IDs to determine the next available ID

---

## Step 2: Parse the User's Input

The user provides a freeform description after `/add-task`. For example:

```
/add-task Fix the login button — it returns a 500 error when clicked on mobile
/add-task Add dark mode toggle to the settings page
/add-task Improve loading performance of the dashboard by lazy-loading charts
```

From the description, determine:

- **Type**: `feature`, `bug`, or `improvement`
- **Title**: A concise title (under 80 chars)
- **Description**: A user-story-style description ("As a [user], I want [X] so that [Y]")
- **Acceptance Criteria**: 3-6 verifiable criteria based on the description

### Type Detection

- **Bug**: mentions errors, broken behavior, crashes, "fix", "doesn't work", "500", "404"
- **Improvement**: mentions performance, refactor, optimize, "improve", "better", "cleanup"
- **Feature**: everything else (new functionality)

---

## Step 3: Ask for Priority

Show the user the new task and ask for priority:

```
New [type] task:

  [ID]: [Title]
  [Description]

  Acceptance Criteria:
  - [criterion 1]
  - [criterion 2]
  - ...

Where should this be prioritized?

1. Highest priority (will be picked up next by developers)
2. After existing todo tasks
3. Custom — I'll specify a priority number
```

Wait for the user's answer.

---

## Step 4: Assign ID and Priority

### ID Assignment

- Look at all existing story IDs in `board.json`
- Assign the next sequential ID: if the highest existing is `US-012`, assign `US-013`
- **Prefix by type:**
  - Features: `US-XXX` (matches existing convention)
  - Bugs: `BUG-XXX`
  - Improvements: `IMP-XXX`
- Each type maintains its own counter (e.g., first bug is `BUG-001` even if `US-015` exists)

### Priority Assignment

Based on user's choice:

1. **Highest priority**: Set `priority: 0` (will sort before priority 1)
2. **After existing todos**: Set priority to `max(existing priorities) + 1`
3. **Custom**: Use the number the user provides

---

## Step 5: Add to Board

1. Read `tasks/board.json` again (in case it changed)
2. Add the new story to the `userStories` array:

```json
{
  "id": "[TYPE-XXX]",
  "title": "[Title]",
  "description": "[Description]",
  "type": "[feature|bug|improvement]",
  "acceptanceCriteria": [
    "[criterion 1]",
    "[criterion 2]",
    "Typecheck passes"
  ],
  "priority": [chosen priority],
  "status": "todo",
  "notes": ""
}
```

3. Write the updated `board.json`

### Rules

- Always include `"Typecheck passes"` as a criterion
- For UI tasks, also include `"Verify in browser"`
- Bugs should include a criterion for "Original bug no longer reproduces"
- Set `status: "todo"`
- Set `notes: ""`

---

## Step 6: Confirm

Tell the user:

```
Added [ID]: [Title] to the board with priority [N].
Type: [feature/bug/improvement]
Status: todo

Run /start-crew → Developers to have it implemented.
```
