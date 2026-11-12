# Vigiloop plugin for Claude Code

Vigiloop's alerts for your projects arrive in your Claude Code session with the evidence, a repro
and what to do, and wake the session even when it is idle. The agent investigates with Vigiloop's
tools and calls `resolve`; Vigiloop re-runs the check and only a passing run closes the alert.

Set up, once per computer, from a terminal:

    claude plugin marketplace add pvarghav/vigiloop-hub && claude plugin install vigiloop@vigiloop
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

## Setting up monitoring

Ask Claude to monitor the project, or run `/vigiloop:monitor-this`. It reads the repo, proposes
checks, writes `vigiloop.config.ts` and check functions in `checks/`, dry-runs, applies after your
yes, and test-runs every function. The same works by hand with the `vigiloop` binary that comes with
the plugin (`~/.claude/plugins/marketplaces/vigiloop/plugin/runtime/vigiloop`):

    vigiloop push                 # what would change
    vigiloop push --apply         # make it so
    vigiloop function test orders --params '{"minOrders": 5}'
    vigiloop secret set SHOP_API_KEY --prompt

Secrets never appear in the config or in an agent's context: the config says where each value
comes from (`secrets: { SHOP_API_KEY: "env:SHOP_API_KEY" }` or `"file:.secrets/shop"`), and the
binary reads it on your machine and sends it straight to Vigiloop, which encrypts it. In CI, set
`VIGILOOP_TOKEN`, `VIGILOOP_API_URL` and `VIGILOOP_HUB_URL` and run `vigiloop push --apply`.

Installed from inside a running session with `/plugin install` instead? Run `/reload-plugins` (or
start a new session) so the plugin's server and hook load.

On a computer with no screen (a server over SSH), sign in from the terminal:

    claude plugin marketplace add pvarghav/vigiloop-hub && claude plugin install vigiloop@vigiloop
    ~/.claude/plugins/marketplaces/vigiloop/plugin/runtime/vigiloop setup <server> <account email>

then approve from the email or the printed link on another device.

Runs on macOS (Apple silicon and Intel), Linux (x64 and arm64) and Windows (x64). Works with your existing
Claude Code install. Vigiloop never sees your Claude credentials and never runs a model itself.

