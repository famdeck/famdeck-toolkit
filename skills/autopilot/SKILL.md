---
name: autopilot
description: "Autonomous work loop — discover tasks from Beads and Relay, pick the highest-priority one, work on it, close it, repeat. Uses Ralph Loop for iteration and BMAD dev-story for implementation. Designed for long-running autonomous work."
---

# Autopilot Work Loop

Work autonomously by discovering and completing tasks from Beads, using Ralph Loop for continuous iteration and BMAD workflows for structured implementation.

## Pre-Flight Checks (run once before starting the loop)

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

### Step 3: Start the Ralph Loop

Construct and start the Ralph Loop with the per-iteration prompt below. Pass user arguments if provided (e.g., `--max-iterations`).

Default invocation:

```
/ralph-loop "<the per-iteration prompt below>" --max-iterations 50 --completion-promise "AUTOPILOT_COMPLETE"
```

If the user specified `--max-iterations N`, use their value instead of 50.
If the user specified `--epic N`, add "Only work on stories labeled epic-N" to the prompt.

---

## Per-Iteration Prompt

Use this exact prompt for `/ralph-loop` (fill in the project path):

~~~
You are an autonomous development agent working on {{project_path}}.
Each iteration: pick one task, implement it, close it.

## Iteration Steps

### 1. Find work

First check for high-priority handoffs:
```bash
bd list --label relay:handoff --status open
```

If handoffs exist, pick the most recent one:
```bash
bd show <handoff-id>
bd update <handoff-id> --status in_progress
```
Read the handoff description for context (branch, decisions, next steps) and resume from where they left off.

If no handoffs, find the next ready task:
```bash
bd ready -n 1
```

If no tasks are available, output: <promise>AUTOPILOT_COMPLETE</promise>

### 2. Claim the task

```bash
bd update <issue-id> --status in_progress
bd show <issue-id>
```

Read the full issue description and acceptance criteria.

### 3. Multi-agent check (if Agent Mail MCP tools are available)

If other agents might be working on this project:
- Check inbox: `fetch_inbox(project_key="{{project_path}}", agent_name="autopilot")`
- Reserve files you plan to edit: `file_reservation_paths(project_key="{{project_path}}", agent_name="autopilot", paths=[...], reason="<issue-id>")`

Skip if you are the only agent.

### 4. Implement

**If the task has a `bmad-story` label** (imported from BMAD):
- This is a BMAD story with acceptance criteria
- Use `/bmad-bmm-dev-story` to implement it — this runs the full structured dev workflow:
  - Story discovery and context loading
  - TDD implementation (write tests first, then implement)
  - Quality validation against acceptance criteria
  - Sprint status updates

**If the task is a regular Beads issue** (no `bmad-story` label):
- Implement directly based on the issue description
- Read relevant code before modifying
- Write tests for your changes
- Run existing tests to verify no regressions

**For all tasks:**
- Create a feature branch: `git checkout -b <issue-id>/<short-description>`
- Commit with the issue ID: `[<issue-id>] <description>`
- Run tests: detect test runner (pytest, vitest, jest, go test) and run it
- Run linter if available (ruff, eslint)

### 5. Quality check

Before closing, verify:
- [ ] Tests pass (run the test suite)
- [ ] Linter clean (run the linter)
- [ ] Changes committed with issue ID in message
- [ ] No untracked files left behind

If tests or linter fail, fix the issues before proceeding.

### 6. Close and clean up

```bash
bd close <issue-id>
```

If you reserved files:
```
release_file_reservations(project_key="{{project_path}}", agent_name="autopilot")
```

If the task was non-trivial in a BMAD project, consider running `/bmad-bmm-code-review` before closing.

### 7. Prepare for next iteration

Run `git checkout main && git pull` to start clean for the next task.

The loop will automatically feed this prompt back for the next iteration.
If there are no more tasks, `bd ready` will return nothing and you will output the completion promise.

## Error Handling

- **Test failures you can't fix:** Leave task in_progress, add comment: `bd update <id> --notes "Tests failing: <details>"`, move to next task
- **Blocked by another task:** `bd dep add <this-id> <blocker-id>`, move to next task
- **Missing context:** Read related issues (`bd show`), check git log, search codebase — investigate, don't guess
- **Stuck after 3 attempts on same issue:** Create a blocker issue: `bd create --title="Blocker: <description>" --type=bug --priority=1`, leave current task in_progress, move on

Do NOT output <promise>AUTOPILOT_COMPLETE</promise> unless `bd ready` returns no tasks AND there are no handoffs. The loop should continue as long as there is work to do.
~~~

---

## Options

- `--max-iterations N` — Override the default 50 iteration limit
- `--epic N` — Only work on stories from Epic N (filter by `epic-N` label)

## Codeman Integration

When running in a Codeman-managed session, use this as the respawn update prompt:

```
Check for relay handoffs and beads ready tasks, then work on the highest priority one. Run /autopilot for the full protocol.
```
