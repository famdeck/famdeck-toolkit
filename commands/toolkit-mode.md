---
name: toolkit-mode
description: "Control how toolkit-init treats individual git repos — set a project to normal (full init), readonly (Atlas only), or ignore (skip). Use when the user wants to exclude or limit a project from toolkit initialization, check its current mode, or list modes across repos."
argument-hint: "[normal | readonly | ignore | --list]"
---

# Project Init Mode

Control how `/toolkit-init` treats a project via `.git/info/toolkit-mode`.

## Modes

| Mode | Effect |
|------|--------|
| **normal** | Full init — Atlas, Relay, Beads, Agent Mail, Serena, BMAD |
| **readonly** | Atlas registry only (no files written). For reference repos |
| **ignore** | Completely skipped by init scans |

## Dispatch

Parse `$ARGUMENTS`:

### No arguments — show current mode

1. Verify `.git/` exists in `$CWD`, error if not a git repo
2. Read `.git/info/toolkit-mode` — if missing, mode is `normal`
3. Display: "**<project-name>**: <mode>" with a brief explanation

### `normal` | `readonly` | `ignore` — set mode

1. Verify `.git/` exists in `$CWD`
2. `normal`: remove `.git/info/toolkit-mode` if it exists
3. `readonly` / `ignore`: `mkdir -p .git/info && echo "<mode>" > .git/info/toolkit-mode`
4. Confirm: "Set **<project-name>** to **<mode>**"

### `--list` — show all projects

1. Scan current directory for git repos (depth 2)
2. Read `.git/info/toolkit-mode` in each
3. Print table of projects with their modes, highlight non-default
