---
name: bmad-import
description: "Import BMAD epics and stories from _bmad-output/ into Beads issues with dependencies, priorities, and traceability. Use when BMAD planning is complete and you need to populate the Beads backlog for execution."
---

# BMAD Story Import

Import BMAD planning artifacts (epics and stories) into Beads issues for execution tracking.

## Prerequisites

- Project must have `_bmad-output/planning-artifacts/epics.md` (run BMAD planning first)
- Beads must be initialized in the project (`bd list` should work)

## Import Process

Follow these steps exactly:

### Step 1: Read the epics file

Read `_bmad-output/planning-artifacts/epics.md` completely. Identify:
- All `## Epic N: Title` sections
- All `### Story N.M: Title` sections within each epic
- Story descriptions and acceptance criteria
- Dependencies between stories (look for references like "after Story X.Y", "depends on", "requires Story")

### Step 2: Check for existing imports

Check if `.beads/bmad-import-manifest.json` exists. If it does, read it — it maps story keys to Beads issue IDs:

```json
{
  "imported_at": "2026-03-01T...",
  "stories": {
    "1.1": { "beads_id": "FD-abc", "title": "Autonomy Readiness Assessment" },
    "1.2": { "beads_id": "FD-def", "title": "Verification Tool Scaffolding" }
  },
  "epics": {
    "1": { "beads_id": "FD-xyz", "title": "Autonomous Development Loop" }
  }
}
```

Skip any story/epic already in the manifest unless `--force` was specified.

### Step 3: Create epic issues

For each `## Epic N: Title` section, create a Beads epic:

```bash
bd create --title="Epic N: <title>" --type=epic --priority=1 --description="<epic description from epics.md>" --labels="bmad-epic,epic-N" --silent
```

Record the returned issue ID in the manifest under `epics.N`.

### Step 4: Create story issues

For each `### Story N.M: Title` section, create a Beads issue:

```bash
bd create \
  --title="Story N.M: <title>" \
  --type=feature \
  --priority=<derived from epic priority and story position> \
  --description="<full story text including As a.../I want.../So that... and all acceptance criteria>" \
  --acceptance="<acceptance criteria section>" \
  --labels="bmad-story,epic-N,story-N.M" \
  --parent=<epic beads_id> \
  --silent
```

**Priority mapping:**
- Epic 1 stories: P1 (bootstrap — must work first)
- Epic 2-3 stories: P2 (core functionality)
- Epic 4-6 stories: P3 (supporting functionality)

Record the returned issue ID in the manifest under `stories.N.M`.

### Step 5: Set up dependencies

After all issues are created, set up dependencies based on:

1. **Explicit dependencies** mentioned in stories (e.g., "Given all Stories 1.1-1.9 are implemented")
2. **Sequential dependencies within an epic** — later stories generally depend on earlier ones within the same epic
3. **Cross-epic dependencies** — Epic 2+ stories may depend on Epic 1 completion

Use `bd dep add <dependent> <dependency>` for each relationship.

**Common dependency patterns from the epics:**
- Story 1.3 (BMAD Import) depends on nothing — can start immediately
- Story 1.4 (Task Discovery) depends on 1.3 (needs issues to discover)
- Story 1.5 (Codeman Sessions) depends on 1.4
- Story 1.6 (Quality Gates) depends on 1.5
- Story 1.7 (Session Completion) depends on 1.5, 1.6
- Story 1.8 (Walkthrough Tests) depends on 1.6, 1.7
- Story 1.9 (Adversarial Review) depends on 1.8
- Story 1.10 (Relay Validation) depends on 1.1-1.9
- Story 1.13 (Loop Runner) depends on 1.4, 1.5, 1.6, 1.7
- Epic 2 stories depend on Epic 1 having basic session management
- Epic 3 stories are independent of Epic 1-2

Read the stories carefully to derive the correct dependency graph. When in doubt, prefer fewer dependencies — don't over-constrain.

### Step 6: Write the manifest

Write the complete manifest to `.beads/bmad-import-manifest.json`:

```json
{
  "imported_at": "<ISO timestamp>",
  "source": "_bmad-output/planning-artifacts/epics.md",
  "stories": {
    "1.1": { "beads_id": "FD-xxx", "title": "..." },
    ...
  },
  "epics": {
    "1": { "beads_id": "FD-yyy", "title": "..." },
    ...
  }
}
```

### Step 7: Report results

Print a summary:
```
BMAD Import Complete
  Epics:   N created, N skipped (already imported)
  Stories: N created, N skipped (already imported)
  Dependencies: N set
  Manifest: .beads/bmad-import-manifest.json
```

## Options

- `--force` — Re-import stories even if they exist in the manifest (creates new issues)
- `--epic N` — Only import stories from Epic N
- `--dry-run` — Show what would be created without creating anything

## Idempotency

The manifest file (`.beads/bmad-import-manifest.json`) tracks all imported stories by their story key (e.g., "1.3"). Re-running the import skips already-imported stories. Use `--force` to override.

## Notes

- The skill reads the epics.md file format as produced by `/bmad-bmm-create-epics-and-stories`
- Story descriptions in Beads include the full acceptance criteria for agent context
- Labels (`bmad-story`, `epic-N`, `story-N.M`) enable filtering and grouping
- Parent relationship (`--parent`) links stories to their epic for `bd epic status`
