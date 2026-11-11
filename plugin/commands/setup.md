---
description: Connect this computer to Vigiloop (sign in once; then its Claude Code sessions show up in Vigiloop)
argument-hint: "<server> <account email> [--strict]"
---

The user wants to connect this computer to Vigiloop. Arguments: $ARGUMENTS

If this computer isn't signed in yet, setup needs the Vigiloop server's address (for example `vigiloop.example.com`) and the email address of the account to join. If they're missing from the arguments and `"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" status` says it isn't signed in, ask the user for them in one short question; don't guess an address.

Run it with the Bash tool, with a 16-minute timeout:

"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" setup $ARGUMENTS

If it signs in, it prints a code and waits. As soon as the code appears, tell the user to approve from the email sent to that account, or by opening the printed /device link on any device (a phone works), only if the code matches.

When it's done, tell the user in two or three sentences what it printed: the mode (global: every Claude Code session on this computer shows up in Vigiloop; strict, with --strict: only sessions they run /vigiloop:enable in), and that no session gets alerts until they pick one on the dashboard, on a project's agent node. No restart is needed.
