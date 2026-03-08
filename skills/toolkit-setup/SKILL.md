---
name: toolkit-setup
description: >
  Use this skill whenever the user wants to install, reinstall, or add any tool from the Famdeck
  ecosystem (Atlas, Relay, Beads, Serena, Agent Mail, Context7) to their Claude Code setup. This
  skill is the correct handler for all Famdeck installation requests — it provides the terminal
  command the user must run outside Claude Code, since `claude plugin install` hangs when called
  from within a session. Invoke for: first-time setup, adding a specific missing tool, running the
  setup wizard, or fixing a broken plugin after an update. Do NOT invoke for: checking status of
  already-installed tools, configuring tools, or installing non-Famdeck software.
---

# Toolkit Setup

The setup script **must run from a regular terminal** (outside Claude Code) because `claude plugin
install` and `claude mcp add` hang when invoked from within a running session.

**Do NOT run the script yourself.** Give the user the command to copy-paste into their terminal:

**Interactive install (recommended):**
```
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/setup.py
```

**Install a specific tool** (when the user named one):
```
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/setup.py --install <tool-name>
```

**Install everything non-interactively:**
```
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/setup.py --non-interactive
```

After the user runs the command and comes back, use `/toolkit-status` to verify the result.
