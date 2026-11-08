# Vigiloop plugin for Claude Code

Vigiloop's alerts for your projects arrive in your Claude Code session with the evidence, a repro
and what to do, and wake the session even when it is idle. The agent investigates with Vigiloop's
tools and calls `resolve`; Vigiloop re-runs the check and only a passing run closes the alert.

Install:

    /plugin install vigiloop --marketplace pvarghav/vigiloop-hub

Setup, once per computer and once per project folder:

1. `/vigiloop:login <server> <account email>`, for example
   `/vigiloop:login vigiloop.example.com you@example.com`. The plugin has no server built in: you
   name the Vigiloop server and the account to join. If that address owns an account there, it
   gets an email to approve this computer. The owner can instead open the `/device` link the
   login prints on any device (a phone works), sign in, and approve there. Approve only if the
   code matches.
2. `/vigiloop:access <project>` in each folder whose alerts this computer should fix.
3. `/vigiloop:status` to check.

On a computer with no screen (a server over SSH), install and sign in from the terminal:

    claude plugin install vigiloop --marketplace pvarghav/vigiloop-hub
    ~/.claude/plugins/marketplaces/vigiloop/plugin/runtime/vigiloop login <server> <account email>

then approve from the email or the printed link on another device.

Runs on macOS (Apple silicon and Intel) and Linux (x64 and arm64). Works with your existing
Claude Code install. Vigiloop never sees your Claude credentials and never runs a model itself.

