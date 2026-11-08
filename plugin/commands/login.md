---
description: Sign this computer in to Vigiloop so its Claude Code sessions can receive and fix alerts
---

Sign this computer in to Vigiloop by running its runtime with the Bash tool, with a 10-minute timeout:

"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" login

It prints a code and waits while the Vigiloop account owner approves the sign-in from an email. As soon as it prints the code, tell the user: check the email from Vigiloop, make sure the code matches, and approve. When it finishes, tell the user what it printed in one or two sentences, and that /vigiloop:access pairs a project folder.
