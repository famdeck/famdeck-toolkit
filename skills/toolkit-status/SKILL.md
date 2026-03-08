---
name: toolkit-status
description: >
  Check whether Famdeck toolkit tools are installed and working: Atlas, Relay, Beads, Serena,
  Agent Mail. Trigger when the user asks what Claude Code tools they have, whether a specific
  Famdeck tool is installed, if their toolkit setup is correct, or wants to verify overall toolkit
  health. Do NOT trigger for checking what issues are open, what projects are registered in Atlas,
  relay tracker config, or the status of non-Famdeck software.
---

# Toolkit Status

Check which toolkit-managed tools are installed and their current state.

## Dispatch

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/scripts/status.py
```

Display the script's output to the user. If any tools are missing, mention that the user can run
`/toolkit-setup` to install them.
