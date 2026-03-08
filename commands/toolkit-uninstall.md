---
name: toolkit-uninstall
description: "Uninstall or remove any tool that the Famdeck toolkit installed — Beads, Atlas, Relay, Serena, Agent Mail, or the toolkit itself. Use whenever the request is about removing, deleting, or cleaning up toolkit-managed Claude Code tools."
argument-hint: "[tool-name | all]"
---

# Toolkit Uninstall

Remove toolkit-managed tools (plugins and MCP servers).

## Dispatch

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/uninstall.py $ARGUMENTS
```

The script shows installed tools and prompts which to remove. Display the results to the user.
