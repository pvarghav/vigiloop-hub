---
description: Update the Vigiloop plugin on this computer to the latest version (use whenever the user asks to update, upgrade or refresh Vigiloop or its plugin)
---

The user wants the Vigiloop plugin updated to the latest version. Do it with the Bash tool, one command at a time. If `claude` isn't found, use "$CLAUDE_CODE_EXECPATH" in its place (it is the running Claude Code).

1. Note the installed version: `claude plugin list --json`, the entry whose id is `vigiloop@vigiloop` (or starts with `vigiloop@`; use that id below).

2. Refresh the marketplace it came from: `claude plugin marketplace update vigiloop`. If there's no marketplace called vigiloop, add it with `claude plugin marketplace add pvarghav/vigiloop-hub`.

3. Update the plugin: `claude plugin update vigiloop@vigiloop`. If it says the plugin isn't installed in that scope, run it again with the scope `claude plugin list --json` showed (`--scope user`, `project` or `local`); if it isn't installed at all, `claude plugin install vigiloop@vigiloop`.

4. Run `claude plugin list --json` again and compare the versions.

Tell the user in one or two sentences: the old and new versions (or that it was already the latest), and that open sessions pick the new version up after `/reload-plugins` or a restart of Claude Code. This session keeps running the old version until then. Don't run setup again: the computer's sign-in carries over. Never delete the plugin's folders or its sign-in (`~/.claude/vigiloop`).

To stop doing this by hand, the user can turn on auto-update: `/plugin`, Marketplaces, vigiloop, Enable auto-update.
