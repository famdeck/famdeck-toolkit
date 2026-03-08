---
name: toolkit-uninstall
description: >
  Invoke to uninstall or remove any tool that the Famdeck toolkit installed — Beads, Atlas, Relay,
  Serena, Agent Mail, or the toolkit itself. This is the only skill that performs removal;
  individual tool skills do not handle their own uninstallation. Use whenever the request is about
  removing, deleting, or cleaning up toolkit-managed Claude Code tools, regardless of how it's
  phrased.
---

# Toolkit Uninstall

Remove toolkit-managed tools (plugins and MCP servers).

## Dispatch

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/uninstall.py $ARGUMENTS
```

Pass any flags or tool names from the user's request through as `$ARGUMENTS`. The script shows
installed tools and prompts which to remove. Display the results to the user.
