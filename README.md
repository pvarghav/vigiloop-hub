# Vigiloop plugin for Claude Code

Vigiloop's alerts for your projects arrive in your Claude Code session with the evidence, a repro
and what to do, and wake the session even when it is idle. The agent investigates with Vigiloop's
tools and calls `resolve`; Vigiloop re-runs the check and only a passing run closes the alert.

Install:

    /plugin install vigiloop --marketplace pvarghav/vigiloop-hub

Setup, once per computer and once per project folder:

1. `/vigiloop:login`: the account owner approves this computer from an email.
2. `/vigiloop:access <project>` in each folder whose alerts this computer should fix.
3. `/vigiloop:status` to check.

Runs on macOS (Apple silicon and Intel). Works with your existing Claude Code install. Vigiloop
never sees your Claude credentials and never runs a model itself.

