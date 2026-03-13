---
name: start-crew
description: "Orchestrate claude-crew agents to work through the task board. Run developer, QA, or design-review agents sequentially. Triggers on: start crew, run developers, run qa, run design review, start developing, start testing."
user-invocable: true
---

# Claude Crew Orchestrator

You are the orchestrator. You do NOT implement anything yourself. Your only job is to spawn sub-agents sequentially and relay their results.

---

## Step 1: Ask What to Run

Ask the user:

```
What crew do you want to run?

1. Developers — pick "todo" stories, implement them, move to "qa"
2. QA Testers — pick "qa" stories, test functionality, move to "design-review" or back to "todo"
3. Design Reviewers — pick "design-review" stories, check visuals, move to "human-review" or back to "todo"
```

Wait for the user's answer.

---

## Step 2: Run the Loop

Based on the user's choice, determine the subagent type:

- **Developers** → subagent type: `developer`
- **QA Testers** → subagent type: `qa`
- **Design Reviewers** → subagent type: `design-review`

Then repeat the following:

1. **Spawn the subagent** using the Agent tool:
   - Set `subagent_type` to the chosen agent type (e.g., `"developer"`)
   - Set a descriptive prompt like: "Execute your task: read the board, pick the next story, and complete it."
   - **Do NOT use isolation/worktree** — agents must work on the actual codebase

2. **Wait for the agent to finish** and read its response.

3. **Check the response:**
   - If the agent says **"NO_MORE_TASKS"** → stop the loop, tell the user all tasks of that type are done.
   - If the agent says **"BLOCKED"** → stop the loop, relay the blocking question/issue to the user. After the user responds, resume the loop.
   - Otherwise → the agent completed a story. Report which story was completed to the user, then go back to step 1 and spawn the next agent.

---

## Rules

- **NEVER implement code yourself.** You are the orchestrator only.
- **NEVER spawn agents in parallel.** Always sequential — one at a time.
- **NEVER skip the user's choice.** Always ask what crew to run first.
- **Report progress** after each agent completes: "Completed [Story ID]: [Title]. Starting next..."
- **If an agent errors or crashes**, report the error to the user and ask how to proceed. Do not retry automatically.

---

## Example Flow

```
You: What crew do you want to run?
     1. Developers  2. QA Testers  3. Design Reviewers

User: 1

You: Starting developer crew...

[Spawns developer subagent]
Agent completes US-001: "Add status field to tasks table"

You: Completed US-001: Add status field to tasks table. Starting next...

[Spawns developer subagent]
Agent completes US-002: "Display status badge on task cards"

You: Completed US-002: Display status badge on task cards. Starting next...

[Spawns developer subagent]
Agent responds: "NO_MORE_TASKS"

You: All todo stories are done! Board status:
     - US-001: qa
     - US-002: qa
     - US-003: qa
     Ready for QA testing.
```
