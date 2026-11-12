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
   List the secrets the functions need by name and where each value comes from on this machine.

4. **Write the files.**
   `vigiloop.config.ts`:
   ```ts
   import { defineConfig } from "@vigiloop/sdk";

   export default defineConfig({
     project: { slug: "shop", name: "Shop", roles: { owner: "dev@example.com" } },
     functions: { orders: "./checks/orders.ts" },
     secrets: { SHOP_API_KEY: "env:SHOP_API_KEY" },
     monitors: [
       { key: "home", type: "http", name: "Homepage", config: { url: "https://shop.example.com/", expect: { status: [200], max_latency_ms: 1500 } } },
       { key: "orders", type: "function", name: "Orders flowing", interval_s: 300, config: { function: "orders", params: { minOrders: 5 } },
         rules: [{ id: "down", type: "consecutive", n: 2 }, { id: "few", type: "threshold", metric: "orders_last_hour", op: "<", value: 5 }] },
     ],
   });
   ```
   A function (`checks/orders.ts`):
   ```ts
   import { defineCheck, z } from "@vigiloop/sdk";

   export default defineCheck({
     description: "Orders keep coming in",
     params: { minOrders: z.number().default(10) },
     secrets: ["SHOP_API_KEY"],
     async run({ params, secrets, metric }) {
       const res = await fetch("https://shop.example.com/api/orders?since=1h", { headers: { authorization: `Bearer ${secrets.SHOP_API_KEY}` } });
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

7. **Prove every function works.** `vigiloop function test <key> --params '{...}'` runs it now with the project's secrets (not recorded, alerts nobody). Fix and push again until each returns what you expect, then report the results.

8. **Hand over.** Say what's being watched and how often; that the checks live in `vigiloop.config.ts` and `checks/` (offer to commit them); that `vigiloop push --apply` (locally or in CI with `VIGILOOP_TOKEN`) re-applies them; and that to have alerts fixed by an agent, they pick a session for the project's agent node on the Vigiloop dashboard.
