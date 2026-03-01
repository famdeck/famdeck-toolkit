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

### Step 2: Scan for existing Beads issues (reconciliation)

Run `bd list` to get ALL existing Beads issues (open, in_progress, and closed). For each issue title:

- If it matches `Story (\d+\.\d+):`, extract the story key (e.g., `1.3`) and record `{story_key → beads_id}`
- If it matches `Epic (\d+):`, extract the epic key (e.g., `1`) and record `{epic_key → beads_id}`

This produces a **pre-existing issue map** used for reconciliation in later steps.

### Step 3: Check manifest and merge with pre-existing issues

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

**Merge the pre-existing issue map into the manifest:**
- If a key exists in BOTH the manifest AND pre-existing issues, keep the manifest entry (it was a prior import)
- If a key exists ONLY in pre-existing issues, add it to the manifest with `"source": "reconciled"`
- Example: `"1.9": { "beads_id": "FD-imt", "title": "Adversarial Review via BMAD Code Review", "source": "reconciled" }`

Skip any story/epic already in the merged manifest unless `--force` was specified.

### Step 4: Create epic issues

For each `## Epic N: Title` section:

1. **Check the merged manifest** — if epic key N already exists, skip creation:
   - If `"source": "reconciled"`, log: `Reconciled Epic N with existing <beads_id>`
   - Otherwise log: `Skipped Epic N (already imported as <beads_id>)`
   - Optionally add `bmad-epic` and `epic-N` labels to the pre-existing issue if missing: `bd label <beads_id> add bmad-epic epic-N`
2. **If not in manifest**, create the epic:

```bash
bd create --title="Epic N: <title>" --type=epic --priority=1 --description="<epic description from epics.md>" --labels="bmad-epic,epic-N" --silent
```

Record the returned issue ID in the manifest under `epics.N`.

### Step 5: Create story issues

For each `### Story N.M: Title` section:

1. **Check the merged manifest** — if story key N.M already exists, skip creation:
   - If `"source": "reconciled"`, log: `Reconciled Story N.M with existing <beads_id>`
   - Otherwise log: `Skipped Story N.M (already imported as <beads_id>)`
   - Optionally add `bmad-story`, `epic-N`, and `story-N.M` labels to the pre-existing issue if missing: `bd label <beads_id> add bmad-story epic-N story-N.M`
2. **If not in manifest**, create the story:

```bash
bd create \
  --title="Story N.M: <title>" \
  --type=feature \
  --priority=<derived from epic priority and story position> \
  --description="Source: _bmad-output/planning-artifacts/epics.md (Story N.M)

<full story text including As a.../I want.../So that... and all acceptance criteria>" \
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

### Step 6: Set up dependencies

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

### Step 7: Write the manifest

Write the complete manifest to `.beads/bmad-import-manifest.json`. Include reconciled entries with their `"source": "reconciled"` marker:

```json
{
  "imported_at": "<ISO timestamp>",
  "source": "_bmad-output/planning-artifacts/epics.md",
  "stories": {
    "1.1": { "beads_id": "FD-xxx", "title": "..." },
    "1.9": { "beads_id": "FD-imt", "title": "Adversarial Review via BMAD Code Review", "source": "reconciled" },
    ...
  },
  "epics": {
    "1": { "beads_id": "FD-yyy", "title": "..." },
    ...
  }
}
```

### Step 8: Report results

Print a summary:
```
BMAD Import Complete
  Epics:   N created, N skipped (already imported), N reconciled (matched existing)
  Stories: N created, N skipped (already imported), N reconciled (matched existing)
  Dependencies: N set
  Manifest: .beads/bmad-import-manifest.json
```

## Options

- `--force` — Re-import stories even if they exist in the manifest (creates new issues)
- `--epic N` — Only import stories from Epic N
- `--dry-run` — Show what would be created without creating anything

## Idempotency

The manifest file (`.beads/bmad-import-manifest.json`) tracks all imported stories by their story key (e.g., "1.3"). Re-running the import skips already-imported stories. Use `--force` to override.

**Reconciliation:** Even without a manifest, the skill scans existing Beads issues for `Story N.M:` and `Epic N:` title patterns. Pre-existing issues are merged into the manifest as `"source": "reconciled"` entries, preventing duplicate creation. This means:
- Deleting the manifest and re-running will NOT create duplicates
- Manually-created issues with proper title format are automatically discovered
- The `--force` flag bypasses both manifest AND reconciliation checks

## Notes

- The skill reads the epics.md file format as produced by `/bmad-bmm-create-epics-and-stories`
- Story descriptions in Beads include the full acceptance criteria for agent context
- Labels (`bmad-story`, `epic-N`, `story-N.M`) enable filtering and grouping
- Parent relationship (`--parent`) links stories to their epic for `bd epic status`
