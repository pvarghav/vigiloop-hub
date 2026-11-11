# Vigiloop plugin for Claude Code

Vigiloop's alerts for your projects arrive in your Claude Code session with the evidence, a repro
and what to do, and wake the session even when it is idle. The agent investigates with Vigiloop's
tools and calls `resolve`; Vigiloop re-runs the check and only a passing run closes the alert.

Set up, once per computer, from a terminal:

    claude plugin install vigiloop --marketplace pvarghav/vigiloop-hub
    claude

then, in that session:

    /vigiloop:setup <server> <account email>

for example `/vigiloop:setup vigiloop.example.com you@example.com`. The plugin has no server built
in: you name it, and the account's address gets an email to approve this computer. The owner can
instead open the `/device` link it prints on any device (a phone works). Approve only if the code
matches.

From then on, every Claude Code session on this computer shows up in Vigiloop, with its title,
folder, computer and app. No session gets alerts until you pick one: on the dashboard, open a
project's agent node and choose the session from the list (grouped by computer and folder).

Two modes, set with `/vigiloop:setup ... --strict` or `/vigiloop:mode global|strict`:

- **global** (default): every session on this computer shows up. `/vigiloop:disable` hides one.
- **strict**: only sessions you run `/vigiloop:enable` in show up.

`/vigiloop:status` shows where things stand.

Installed from inside a running session with `/plugin install` instead? Run `/reload-plugins` (or
start a new session) so the plugin's server and hook load.

On a computer with no screen (a server over SSH), sign in from the terminal:

    claude plugin install vigiloop --marketplace pvarghav/vigiloop-hub
    ~/.claude/plugins/marketplaces/vigiloop/plugin/runtime/vigiloop setup <server> <account email>

then approve from the email or the printed link on another device.

Runs on macOS (Apple silicon and Intel) and Linux (x64 and arm64). Works with your existing
Claude Code install. Vigiloop never sees your Claude credentials and never runs a model itself.

