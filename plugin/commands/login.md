---
description: Sign this computer in to a Vigiloop server so its Claude Code sessions can receive and fix alerts
argument-hint: "<server> <account email>"
---

The user wants to sign this computer in to Vigiloop. Arguments: $ARGUMENTS

It needs two things: the Vigiloop server's address (for example `vigiloop.example.com`) and the email address of the account to join. If either is missing from the arguments, ask the user for it in one short question and wait; don't guess or fill in an address yourself.

Then run the runtime with the Bash tool, with a 16-minute timeout, passing both:

"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop" login <server> <account email>

It checks the server, prints a code, and waits while the account owner approves. As soon as the code appears, tell the user: approve it from the email sent to that account, or open the printed /device link on any device (a phone works) and sign in there; either way, only if the code matches. When it finishes, tell the user what it printed in one or two sentences, and that /vigiloop:access pairs a project folder.

On a computer without Claude Code open (a server over SSH, say), the same command runs in a plain terminal.
