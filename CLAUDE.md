# claude-crew

A reusable set of Claude Code skills for orchestrating AI agents to build features autonomously.

## What is this?

claude-crew is a task board system where AI agents work through user stories sequentially:
1. **`/build-prd`** — Generate a PRD from a feature description
2. **`/build-board`** — Convert the PRD into `tasks/board.json` (a structured task board)
3. **`/start-crew`** — Orchestrate sub-agents to work through the board

## Agent Types

- **Developer** — Picks up "todo" stories, implements them (TDD for logic, browser verify for UI), commits, moves to "qa"
- **QA Tester** — Picks up "qa" stories, tests functionality via Playwright, moves to "design-review" or back to "todo"
- **Design Reviewer** — Picks up "design-review" stories, checks visual quality via screenshots, moves to "human-review" or back to "todo"

## How to Use in Any Project

Copy the `.claude/skills/` folder into your project's `.claude/` directory. Then use the slash commands:

```
/build-prd          — create a PRD
/build-board        — create board.json from the PRD
/start-crew         — run agents (choose: developers, QA, or design review)
/add-task           — add a new task (feature, bug, improvement) to the board
/review-task        — approve or reject tasks in human-review
```

## File Structure

```
tasks/
  prd-[feature].md    — PRD document
  board.json          — Task board with user stories and statuses
  progress.txt        — Progress log and codebase patterns
```

## Story Statuses

`todo` → `in-progress` → `qa` → `design-review` → `human-review` → `done`
