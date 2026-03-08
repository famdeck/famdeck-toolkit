---
name: toolkit-setup
description: "Use when the user wants to install, reinstall, or add any Famdeck tool (Atlas, Relay, Beads, Serena, Agent Mail, Context7) to their Claude Code setup — first-time setup, adding a missing tool, running the setup wizard, or fixing a broken plugin after an update."
argument-hint: "[tool-name | all]"
---

# Toolkit Setup

The setup script **must run from a regular terminal** (outside Claude Code) because `claude plugin install` and `claude mcp add` hang when invoked from within a running session.

**Do NOT run the script yourself.** Instead, give the user the appropriate command to copy-paste:

**Interactive install:**
```
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/setup.py
```

**Install a specific tool** (when `$ARGUMENTS` contains a tool name):
```
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/setup.py --install $ARGUMENTS
```

**Install all standard tools** (when `$ARGUMENTS` is `all`):
```
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/setup.py --non-interactive
```

After the user runs the command, use `/toolkit-status` to verify the result.
