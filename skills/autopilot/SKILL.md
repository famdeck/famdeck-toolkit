---
name: autopilot
description: "Autonomous work loop — discover tasks from Beads and Relay, pick the highest-priority one, work on it, close it, repeat. Uses Agent tool for per-task isolation and BMAD dev-story for implementation. Designed for long-running autonomous work."
---

# Autopilot Work Loop

Work autonomously by discovering and completing tasks from Beads. Each task runs in an isolated Agent subprocess (equivalent to a Codeman session in the target architecture).

## Architecture

```
autopilot (this skill)     ← outer loop: task discovery + lifecycle
  └─ Agent tool (per task) ← inner session: isolated execution per task
       └─ BMAD dev-story   ← implementation methodology (if BMAD story)
```

- **Outer loop**: This skill manages task cycling — pick, dispatch, close, repeat
- **Agent tool**: Each task runs in a subagent with its own context window (maps to Codeman sessions in target architecture)
- **BMAD dev-story**: Structured implementation inside the agent session

## Pre-Flight Checks (run once)

### Step 0: Verify BMAD planning state

If the project has a `_bmad/` directory:

1. Check `_bmad-output/planning-artifacts/` for:
   - `*prd*` — PRD exists?
   - `*architecture*` — Architecture exists?
   - `*epic*` — Epics/stories exist?

2. If any are missing, tell the user what's needed and stop:
   - No PRD → suggest `/bmad-bmm-create-prd`
   - No architecture → suggest `/bmad-bmm-create-architecture`
   - No epics → suggest `/bmad-bmm-create-epics-and-stories`

3. If all exist → proceed.

If no `_bmad/` directory → not a BMAD project, skip this step.

### Step 1: Ensure Beads backlog has work

Run `bd ready` to check for available tasks.

If Beads is empty AND `.beads/bmad-import-manifest.json` does NOT exist AND BMAD epics exist:
- Run `/bmad-import` to populate the backlog from BMAD planning artifacts
- After import, run `bd ready` again to confirm tasks are available

If Beads is still empty after import attempt → tell user there's nothing to work on and stop.

### Step 2: Check for handoffs

```bash
bd list --label relay:handoff --status open
```

If handoffs exist, note the most recent one — it will be prioritized in the loop.

## Task Loop

After pre-flight checks pass, enter the task loop. Repeat until no work remains:

### For each iteration:

#### 1. Find the next task

Check for handoffs first (highest priority):
```bash
bd list --label relay:handoff --status open
```

If no handoffs, find the next ready task:
```bash
bd ready -n 1
```

If no tasks available → **stop the loop**. Report summary and exit.

#### 2. Claim it

```bash
bd update <issue-id> --status in_progress
bd show <issue-id>
```

Read the full description and acceptance criteria.

#### 3. Dispatch to Agent

Launch the task in an isolated Agent subprocess using the **Agent tool**. This gives each task its own context window and prevents context bloat in the main session.

Use `subagent_type: "general-purpose"` and construct the prompt based on the task type:

**For BMAD stories** (has `bmad-story` label):

```
Agent prompt:
"You are working on project /path/to/project.
Task: [issue-id] [title]

[Full issue description and acceptance criteria from bd show]

Implementation instructions:
1. Create a feature branch: git checkout -b [issue-id]/[short-name]
2. Use /bmad-bmm-dev-story to implement this story — it runs the full structured dev workflow with TDD, quality validation, and acceptance criteria checking.
3. After implementation, run tests and linter to verify quality.
4. Commit all changes with [issue-id] in the commit message.
5. Push the branch: git push -u origin [branch-name]

When done, report: what was implemented, test results, files changed."
```

**For regular Beads issues** (no `bmad-story` label):

```
Agent prompt:
"You are working on project /path/to/project.
Task: [issue-id] [title]

[Full issue description from bd show]

Implementation instructions:
1. Create a feature branch: git checkout -b [issue-id]/[short-name]
2. Read relevant code before modifying.
3. Write tests for your changes.
4. Implement the changes.
5. Run tests and linter to verify.
6. Commit with [issue-id] in the message.
7. Push the branch: git push -u origin [branch-name]

When done, report: what was implemented, test results, files changed."
```

**For handoffs:**

```
Agent prompt:
"You are resuming work from a handoff on project /path/to/project.
Handoff: [issue-id]

[Full handoff description including branch, decisions, next steps]

Resume from where they left off. Follow the next steps described in the handoff.
When done, report what was accomplished."
```

**Agent options:**
- Use `run_in_background: false` — wait for the agent to complete before proceeding
- The agent has access to all tools (Bash, Read, Write, Edit, Grep, etc.)

#### 4. Process the result

When the Agent returns:
- Review its report (what was implemented, test results, files changed)
- If the agent reports success → proceed to close
- If the agent reports failure → add notes to the issue and move to the next task:
  ```bash
  bd update <issue-id> --notes "Agent failed: <details from agent report>"
  ```

#### 5. Close the task

```bash
bd close <issue-id>
```

Clean up:
```bash
git checkout main && git pull
```

#### 6. Continue the loop

Go back to step 1 (Find the next task). The loop continues until `bd ready` returns no tasks and no handoffs exist.

## When to stop

- `bd ready` returns no tasks AND no handoffs exist → report summary and exit
- User explicitly tells you to stop
- An unrecoverable error occurs (report it and exit)

## Error Handling

- **Agent fails on a task:** Add notes to the issue, leave it in_progress, move to the next task
- **No test runner found:** Agent should report this; note it and continue
- **Blocked by another task:** `bd dep add <this-id> <blocker-id>`, move to next task
- **Stuck after 3 attempts on same issue:** Create a blocker issue: `bd create --title="Blocker: <description>" --type=bug --priority=1`, move on

## Loop Summary

After all tasks are done (or the loop stops), print a summary:

```
Autopilot Summary
  Tasks completed: N
  Tasks failed: N (left in_progress with notes)
  Tasks skipped: N (blocked/deferred)
  Remaining ready: N
```

## Options

- `--epic N` — Only work on stories from Epic N (filter by `epic-N` label in `bd ready`)
- `--max-tasks N` — Stop after completing N tasks (safety limit)
- `--dry-run` — Show what tasks would be picked without executing them

## Codeman Integration (future)

When Codeman sessions are available (famdeck-plugin Epic 2), the Agent tool dispatch is replaced by Codeman session orchestration:
- `POST /api/sessions` instead of Agent tool
- SSE monitoring instead of waiting for Agent return
- Ralph Loop runs INSIDE Codeman sessions for iterative refinement
- The outer task loop structure remains the same
