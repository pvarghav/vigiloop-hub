---
description: Connect this computer and this folder to Vigiloop in one step (sign in if needed, then pair the folder's project)
argument-hint: "<server> <account email> [--project=slug]"
---

The user wants this folder's Claude Code sessions to receive Vigiloop alerts. Arguments: $ARGUMENTS

If this computer isn't signed in yet, setup needs the Vigiloop server's address (for example `vigiloop.example.com`) and the email address of the account to join. If they're missing from the arguments and `"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" status` says it isn't signed in, ask the user for them in one short question; don't guess an address.

Run it with the Bash tool from this project's folder, with a 16-minute timeout:

"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" setup $ARGUMENTS

If it signs in, it prints a code and waits. As soon as the code appears, tell the user to approve from the email sent to that account, or by opening the printed /device link on any device (a phone works), only if the code matches. If it exits listing several projects, ask the user which one this folder is for and run it again with --project=<slug>. When it's done, tell the user in one or two sentences: this folder is paired (no restart needed), and this session takes alerts only once they put it on call on the Vigiloop dashboard (the agent node's Sessions list, link printed above). Nothing reaches a session until then.
