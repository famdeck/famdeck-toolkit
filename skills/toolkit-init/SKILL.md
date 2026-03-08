---
name: toolkit-init
description: >
  Bulk-initialize multiple git repositories for the Famdeck ecosystem — registers them in Atlas,
  configures Relay trackers, sets up Beads, Agent Mail, and Serena. Trigger when the user wants
  to scan a directory of repos and onboard them all at once, or run the toolkit init command.
  Do NOT trigger for initializing a single project individually, creating a new git repo, or
  setting up non-Famdeck tools.
---

# Batch Project Init

Scan the current directory for git repositories and initialize each for the Famdeck ecosystem
(Atlas, Relay, Beads, Serena, Agent Mail).

## Dispatch

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/init.py $ARGUMENTS
```

Pass any flags from the user's request through as `$ARGUMENTS`. Display the script's output.

## Options

| Flag | Effect |
|------|--------|
| `--depth=N` | Scan depth for git repos (default: 2) |
| `--atlas-only` | Only register in Atlas |
| `--relay-only` | Only configure Relay trackers |
| `--no-bmad` | Skip BMAD installation |
| `--non-interactive` | No prompts, use defaults |
