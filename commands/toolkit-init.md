---
name: toolkit-init
description: "Bulk-initialize multiple git repositories for the Famdeck ecosystem — registers them in Atlas, configures Relay, sets up Beads, Agent Mail, Serena. Use when the user wants to scan a directory and onboard all repos at once, or run toolkit-init."
argument-hint: "[--depth=N] [--atlas-only | --relay-only | --no-bmad | --non-interactive]"
---

# Batch Project Init

Scan the current directory for git repositories and initialize each one for the Famdeck ecosystem (Atlas, Relay, Beads, Serena, Agent Mail).

## Dispatch

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/init.py $ARGUMENTS
```

Pass `$ARGUMENTS` through verbatim. Display the script's output to the user.

## Options

| Flag | Effect |
|------|--------|
| `--depth=N` | Scan depth for git repos (default: 2) |
| `--atlas-only` | Only register projects in Atlas |
| `--relay-only` | Only configure Relay trackers |
| `--no-bmad` | Skip BMAD installation |
| `--non-interactive` | No prompts, use defaults |
