---
description: Let this folder's Claude Code sessions receive Vigiloop alerts for a project (or list the projects)
argument-hint: "[project slug] [--role=oncall|interactive] [--remove]"
---

Run the Vigiloop runtime with the Bash tool from this project's folder:

"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" access $ARGUMENTS

With no arguments it lists the Vigiloop projects this computer may pair and what this folder is paired with now; with a project slug it pairs this folder, so sessions here receive that project's alerts. --role=oncall marks sessions here as the designated unattended ones (alerts prefer them); the default, interactive, is a person's own session. Tell the user the result in one or two sentences. If they gave no project and there is exactly one, ask whether to pair it.
