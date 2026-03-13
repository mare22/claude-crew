---
name: build-board
description: "Convert a PRD into tasks/board.json with prioritized user stories for the claude-crew agent system. Triggers on: build board, create board, convert prd to board, board from prd."
user-invocable: true
---

# Board Builder

Converts a PRD into `tasks/board.json` — a structured task board that claude-crew agents use for autonomous execution.

---

## The Job

1. Read the PRD file (user provides path or you find it in `tasks/prd-*.md`)
2. Convert each user story into a board entry with status tracking
3. Save to `tasks/board.json`
4. Initialize `tasks/progress.txt` if it doesn't exist

**Important:** Do NOT start implementing. Just create the board.

---

## Output Format

```json
{
  "project": "[Project Name]",
  "prdSource": "tasks/prd-[feature-name].md",
  "description": "[Feature description from PRD]",
  "userStories": [
    {
      "id": "US-001",
      "title": "[Story title]",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": [
        "Criterion 1",
        "Criterion 2",
        "Typecheck passes"
      ],
      "priority": 1,
      "status": "todo",
      "notes": ""
    }
  ]
}
```

### Status Values

| Status | Meaning |
|---|---|
| `todo` | Ready for development |
| `in-progress` | Developer is working on it |
| `blocked` | Waiting for human decision |
| `qa` | Ready for QA testing |
| `design-review` | Ready for visual/design review |
| `human-review` | Passed QA + design, waiting for human approval |
| `done` | Human approved |

All stories start with `"status": "todo"`.

---

## Story Size: The Number One Rule

**Each story must be completable in ONE agent iteration (one context window).**

Each agent spawns fresh with no memory of previous work. If a story is too big, the agent runs out of context before finishing and produces broken code.

### Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

### Too big (split these):
- "Build the entire dashboard" — Split into: schema, queries, UI components, filters
- "Add authentication" — Split into: schema, middleware, login UI, session handling
- "Refactor the API" — Split into one story per endpoint or pattern

**Rule of thumb:** If you cannot describe the change in 2-3 sentences, it is too big.

---

## Story Ordering: Dependencies First

Stories execute in priority order. Earlier stories must not depend on later ones.

**Correct order:**
1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Dashboard/summary views that aggregate data

**Wrong order:**
1. UI component (depends on schema that doesn't exist yet)
2. Schema change

---

## Acceptance Criteria: Must Be Verifiable

Each criterion must be something an agent can CHECK, not something vague.

### Good criteria (verifiable):
- "Add `status` column to tasks table with default 'pending'"
- "Filter dropdown has options: All, Active, Completed"
- "Clicking delete shows confirmation dialog"
- "Typecheck passes"
- "Tests pass"

### Bad criteria (vague):
- "Works correctly"
- "User can do X easily"
- "Good UX"
- "Handles edge cases"

### Always include as final criterion:
```
"Typecheck passes"
```

For stories with testable logic, also include:
```
"Tests pass"
```

For stories that change UI, also include:
```
"Verify in browser"
```

---

## Conversion Rules

1. **Each user story becomes one JSON entry**
2. **IDs**: Sequential (US-001, US-002, etc.)
3. **Priority**: Based on dependency order, then document order (1 = highest)
4. **All stories**: `status: "todo"` and empty `notes`
5. **Always add**: "Typecheck passes" to every story's acceptance criteria
6. **UI stories**: Also add "Verify in browser"

---

## Splitting Large PRDs

If a PRD has big features, split them:

**Original:**
> "Add user notification system"

**Split into:**
1. US-001: Add notifications table to database
2. US-002: Create notification service for sending notifications
3. US-003: Add notification bell icon to header
4. US-004: Create notification dropdown panel
5. US-005: Add mark-as-read functionality
6. US-006: Add notification preferences page

Each is one focused change that can be completed and verified independently.

---

## Initialize Progress File

After writing `board.json`, create `tasks/progress.txt` if it doesn't exist:

```
# Progress Log
# This file tracks work done by claude-crew agents.
# Each agent appends their progress after completing a story.
# The "Codebase Patterns" section is read by future agents to learn from past work.

## Codebase Patterns
(Agents: add patterns, gotchas, and useful context here as you discover them)

---

```

---

## Example

**Input PRD excerpt:**
```markdown
## User Stories

### US-001: Add status field to database
As a developer, I need to store task status in the database.
- Add status column: 'pending' | 'in_progress' | 'done'
- Generate and run migration

### US-002: Display status badge on task cards
As a user, I want to see task status at a glance.
- Colored badge on each card
- Badge colors: gray=pending, blue=in_progress, green=done
```

**Output board.json:**
```json
{
  "project": "TaskApp",
  "prdSource": "tasks/prd-task-status.md",
  "description": "Task Status Feature - Track task progress with status indicators",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add status field to tasks table",
      "description": "As a developer, I need to store task status in the database.",
      "acceptanceCriteria": [
        "Add status column: 'pending' | 'in_progress' | 'done' (default 'pending')",
        "Generate and run migration successfully",
        "Typecheck passes"
      ],
      "priority": 1,
      "status": "todo",
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "Display status badge on task cards",
      "description": "As a user, I want to see task status at a glance.",
      "acceptanceCriteria": [
        "Each task card shows colored status badge",
        "Badge colors: gray=pending, blue=in_progress, green=done",
        "Typecheck passes",
        "Verify in browser"
      ],
      "priority": 2,
      "status": "todo",
      "notes": ""
    }
  ]
}
```

---

## Checklist Before Saving

- [ ] Each story is completable in one iteration (small enough)
- [ ] Stories are ordered by dependency (schema → backend → UI)
- [ ] Every story has "Typecheck passes" as criterion
- [ ] UI stories have "Verify in browser" as criterion
- [ ] Acceptance criteria are verifiable (not vague)
- [ ] No story depends on a later story
- [ ] All statuses set to "todo"
- [ ] `tasks/progress.txt` initialized (if doesn't exist)
- [ ] Saved to `tasks/board.json`
