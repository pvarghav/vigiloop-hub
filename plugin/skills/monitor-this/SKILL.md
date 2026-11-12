---
name: monitor-this
description: Set up Vigiloop monitoring for the current project as code. Reads the repo, proposes checks (HTTP, heartbeats for cron jobs, webhook expectations, cloud functions for business logic), writes vigiloop.config.ts and check functions, wires secrets by reference without ever reading them, dry-runs, applies after the user agrees, and test-runs each function. Use when the user asks to monitor this project, add checks or alerts, or set up Vigiloop here.
---

# Set up Vigiloop monitoring for this project

You set up monitoring as code: a `vigiloop.config.ts` at the project root plus check functions in `checks/`, applied with the `vigiloop` binary. Everything you create is reviewable in the repo and reproducible with one command.

The binary: `"${CLAUDE_PLUGIN_ROOT}/runtime/vigiloop"` (below: `vigiloop`). Run `vigiloop --help` for every command.

## Ground rules

- **Never see a secret value.** Don't print, cat, echo or grep files that hold secrets (`.env*`, `*.pem`, credential files), and never ask the user to paste a secret into the chat. Secrets are referenced in the config as `"env:VAR"` or `"file:path"`; the binary reads the value on this machine and sends it straight to Vigiloop. To check a variable exists without seeing it: `[ -n "$VAR" ] && echo set || echo missing`. To make values from an env file available to one command without printing them: `(set -a; . ./.env.local; set +a; vigiloop push --apply)`.
- **Nothing changes without the user's yes.** Propose the plan, show the dry run, then apply.
- **Don't choose who gets alerts.** Which Claude Code session receives a project's alerts is picked by a person on the Vigiloop dashboard. Tell them where; don't try to set it.

## Steps

1. **Check the connection.** `vigiloop status`. If it isn't signed in, or says the sign-in can't set things up, ask the user for the Vigiloop server and their account email and run `vigiloop setup <server> <email>` (it waits for them to approve by email or on the printed link). An existing `vigiloop.config.ts` means the project is already set up: read it and extend it instead of starting over.

2. **Understand what matters in production.** Read the README, deploy config (wrangler, vercel, Dockerfile, CI), routes and API handlers, scheduled jobs, webhook handlers, and anything that takes money or signs people in. Find the public URLs (from config, not by guessing).

3. **Propose a short plan** and wait for the user. For each check: what it watches, the type, how often, the condition that alerts, and who hears (email by default; the agent action hands the alert to a Claude Code session). Keep it to what would really hurt if it broke. Check types:
   - `http`: an endpoint answers (status, body text, latency).
   - `heartbeat`: a cron job or worker pings `POST /in/heartbeat/<id>` each run; alerts when it goes quiet.
   - `webhook_expect`: an external sender (Stripe, GitHub…) keeps sending an event; needs a source with a signing secret.
   - `function`: your own TypeScript check for anything an HTTP check can't express (a logged-in flow, "orders in the last hour", a third-party API's state).
   List the secrets the functions need by name and where each value comes from on this machine. Settings that aren't secret (base URLs, thresholds, regions, account ids) go in `vars`, written in the config.

4. **Write the files.**
   `vigiloop.config.ts`:
   ```ts
   import { defineConfig } from "@vigiloop/sdk";

   export default defineConfig({
     project: { slug: "shop", name: "Shop", roles: { owner: "dev@example.com" } },
     functions: { orders: "./checks/orders.ts" },
     secrets: { SHOP_API_KEY: "env:SHOP_API_KEY" },
     vars: { SHOP_API_URL: "https://shop.example.com/api" },
     monitors: [
       { key: "home", type: "http", name: "Homepage", config: { url: "https://shop.example.com/", expect: { status: [200], max_latency_ms: 1500 } } },
       { key: "orders", type: "function", name: "Orders flowing", interval_s: 300, config: { function: "orders", params: { minOrders: 5 } },
         rules: [{ id: "down", type: "consecutive", n: 2 }, { id: "few", type: "threshold", metric: "orders_last_hour", op: "<", value: 5 }] },
     ],
     groups: [{ title: "Storefront", monitors: ["home", "orders"] }],
   });
   ```
   A function (`checks/orders.ts`):
   ```ts
   import { defineCheck, z } from "@vigiloop/sdk";

   export default defineCheck({
     description: "Orders keep coming in",
     params: { minOrders: z.number().default(10) },
     secrets: ["SHOP_API_KEY"],
     vars: ["SHOP_API_URL"],
     async run({ params, secrets, vars, metric }) {
       const res = await fetch(`${vars.SHOP_API_URL}/orders?since=1h`, { headers: { authorization: `Bearer ${secrets.SHOP_API_KEY}` } });
       if (!res.ok) throw new Error(`orders API returned ${res.status}`);
       const { count } = (await res.json()) as { count: number };
       metric("orders_last_hour", count);
       if (count < params.minOrders) return { status: "degraded", summary: `Only ${count} orders in the last hour` };
     },
   });
   ```
   Functions: return nothing for ok, `{ status: "degraded" | "fail", summary }` otherwise, or throw to fail; `metric()` values feed `threshold` and `delta` conditions; `console.log` lands in the run's logs. They may import only `@vigiloop/sdk` and their own relative files (no npm packages), use `fetch`, and get 10 s of CPU and 50 requests per run. Write a `summary` a person on call understands.

5. **Secrets.** For each one, check the variable exists (without printing it). If one is missing, tell the user the exact name and ask them to either export it (or add it to the env file you load) or set it themselves with `! vigiloop secret set NAME --prompt --project <slug>`. Push reports any secret a function needs that still has no value.

6. **Dry run, then apply.** `vigiloop push` shows what would change; show it to the user. After they agree: `vigiloop push --apply`.

7. **Prove every function works.** `vigiloop function test <key> --params '{...}'` runs it now with the project's variables and secrets (not recorded, alerts nobody). Fix and push again until each returns what you expect, then report the results.

8. **Hand over.** Say what's being watched and how often; that the checks live in `vigiloop.config.ts` and `checks/` (offer to commit them); that `vigiloop push --apply` (locally or in CI with `VIGILOOP_TOKEN`) re-applies them; and that to have alerts fixed by an agent, they pick a session for the project's agent node on the Vigiloop dashboard.

## Reference

Everything below is what `vigiloop.config.ts` accepts. You shouldn't need anything else.

**Project:** `project: { slug, name, roles?: { owner: "a@b.com", oncall: "c@d.com" } }`. `slug` is lowercase letters, digits, `-` and `_`. Role addresses get `notify_role` actions and can sign in to the dashboard read-only (the account owner edits).

**Every monitor:** `{ key, type, name, config, interval_s?: 60 (30 to 86400), enabled?: true, muted?: false, rules?: [...] }`. `key` is stable within the project (the monitor's id derives from it). Durations are a number of ms or a string like `"90s"`, `"10m"`, `"25h"`, `"7d"`.

**Monitor types and their config:**
- `http`: `{ url, method?: "GET", headers?: {}, body?: "", timeout_ms?: 10000, follow_redirects?: true, expect?: { status?: [200], body_contains?: "", body_not_contains?: "", header?: { name: value }, max_latency_ms?: 1500 } }`. Without `expect.status`, any status below 400 passes. Above `max_latency_ms` a run is degraded. Metrics: `latency_ms`, `ttfb_ms`, `status_code`.
- `heartbeat`: `{ max_gap: "25h", grace_after_create?: "1h", active_hours?: { tz: "Europe/Stockholm", from: "08:00", to: "18:00" } }`. The job sends `POST https://<api>/in/heartbeat/<monitor id>` after each successful run (no auth, no body). `push` prints each heartbeat's exact URL, and `vigiloop monitors` shows it and the last ping. Put the URL in the job (an env var is best) and ping only after the work succeeded, so a crash shows up as silence. Set `max_gap` to the schedule plus slack (daily job: `"25h"`). Don't send test pings unless the user asks: a ping counts as a run. Metric: `age_s` (seconds since the last ping).
- `webhook_expect`: `{ source: "<source name>", event_type: "invoice.paid", max_gap: "2h", grace_after_create?, active_hours? }`. Expects that event from that sender at least every `max_gap`. Needs a source (below), whose ingest URL `push` prints; the user registers it at the sender.
- `function`: `{ function: "<key under functions>", params?: {}, timeout_ms?: 30000 (1000 to 60000) }`.

**Webhook sources:** `sources: [{ name: "stripe", signature_scheme: "stripe" | "github" | "hmac_sha256" | "none", secret: "env:STRIPE_WEBHOOK_SECRET", event_type_path?: "type", idem_key_path?: "id" }]`. The signing secret is a reference like any other.

**Rules** (when nothing is set: `consecutive` with `n: 2` for http and function, `absence` for heartbeat and webhook_expect, all emailing the owner). Every rule has `{ id, type, actions?: [{ kind: "email" }], high_risk?: false }`.
- `consecutive`: `{ n: 2 }`. Fires after n failed runs in a row. Runner errors (a broken check, a timeout) neither count nor reset it.
- `threshold`: `{ metric, op: ">" | ">=" | "<" | "<=", value }`. Fires while a run's metric crosses it.
- `absence`: `{ max_gap? }`. Fires when a heartbeat or webhook goes quiet. `max_gap` overrides the monitor's.
- `delta`: `{ metric, pct?: 50, abs?: 100, direction?: "up" | "down" | "any" }`. Fires when a metric moves that much from the previous run.
- `high_risk: true` keeps agents out: a person handles it.
- An alert notifies once when it opens and once when it resolves; runs in between send nothing. For problems that can persist (a credit balance running low, a certificate nearing expiry), add `remind_every: "24h"` to notify again while it lasts (people only; agents already have the alert). For values that hover around a threshold, `clear_after: 2` resolves only after two passing runs in a row.
- Example, a balance a person tops up: `{ id: "low-balance", type: "threshold", metric: "balance_usd", op: "<", value: 20, actions: [{ kind: "email" }], remind_every: "24h", clear_after: 2 }`.

**Actions:** `{ kind: "email", to?: ["a@b.com"] }` (default: the account owner), `{ kind: "notify_role", role: "oncall" }`, `{ kind: "webhook", url }` (POSTs the alert as JSON), `{ kind: "agent" }` (hands the alert to the Claude Code session picked for the project on the dashboard; it fixes the problem and Vigiloop re-runs the check before closing).

**Secrets:** `secrets: { NAME: "env:VAR" | "file:path" }`. NAME is uppercase letters, digits and `_`. A function gets only the names it lists in `defineCheck({ secrets })`. `push` reports each one as created, updated or unchanged, and names any a function needs that has no value.

**Variables:** `vars: { NAME: "value" }` (strings, numbers or booleans; stored as strings). Plain project settings that every function in the project reads as `ctx.vars.NAME`; a function lists the ones it can't run without in `defineCheck({ vars })`, and a run without one is a runner error. Values are stored as written and shown on the dashboard, so never put a credential here (push warns about names like `*_KEY` or `*_TOKEN`). Where an input goes: differs per monitor → `params`; one setting for the project that a person may want to change without touching code → `vars`; anything secret → `secrets`. A person can change a variable on the dashboard or with `vigiloop var set NAME VALUE`; the next push sets variables the config lists back to the config's value.

**Groups:** `groups: [{ title: "Checkout", monitors: ["cart", "payments"] }]` draws a titled outline on the dashboard canvas around those monitors' cards (their checks, conditions, and any action only they use). Group monitors that belong to one user-facing flow or one system, so a person can see at a glance what a failure touches. Listing `groups` makes the config's groups exactly these (`[]` removes them); leaving it out keeps the ones an earlier push set. Groups a person draws on the dashboard are never touched.

**Commands:** `vigiloop push [--apply] [--prune]` (`--prune` deletes monitors, functions and variables the config no longer lists), `vigiloop function test KEY [--params JSON]`, `vigiloop functions`, `vigiloop monitors`, `vigiloop var set|list|delete`, `vigiloop secret set|list|delete`. All take `--project <slug>` when run outside the project folder.
