# POS Software (artifact app) — Vercel port

A Vercel-deployable form of the **actual POS Software artifact app**, ported
from the saved Version 1 copy at
`workspace/pos-backups/pos-software-version-1-20261031/` (2026-10-31).
It is the same React client and the same server actions — not the standalone
v2 rebuild that was deployed earlier and rejected.

**Nothing here touches the live Muse artifact or the backup.** The backup was
copied once, read-only; all work happened in this directory. No business data
was copied: the backup's `app.db` was deliberately excluded, so this port
starts with **fresh empty data** on first deploy.

## What was preserved exactly

- The whole client: `client/src/App.tsx` (all screens/flows), `api.ts`,
  `main.tsx`, `theme.css` — only the three `@hatch/space-sdk/client` import
  lines were swapped to the local `./space-client` shim.
- All **43** server actions in `server/src/actions.ts` (39 from the backup —
  the brief said 40; the code defines 39, see "Count note" below — plus 3
  added 2026-11-08: `requestPasswordReset`, `resetPasswordWithCode`,
  `adminResetAllData`; see `FORGOT-PASSWORD-WIPE-NOTES.md` — plus 1 added
  2026-11-09: `restockProduct`; see `RESTOCK-NOTES.md`) with identical zod
  request/response schemas, handler logic, plan rules (trial/basic/standard/
  pro, prices 0/1999/3999/6999), PBKDF2 password hashing (`s2$` format),
  30-day DB sessions, credit guards, money/P&L computations, and the
  one-time Super Admin setup semantics (setup card only while no *usable*
  Super Admin exists, including reclaiming a disabled legacy pilot row).
- The drizzle schema (`server/src/schema.ts`, 15 tables) and all 6 drizzle
  migrations (0001–0006) verbatim, embedded into `server/src/migrations.ts`.

## Added afterwards: the two post-backup 2026-10-31 build features

The Version 1 backup predates the 2026-10-31 main-artifact build, so these
two features were re-implemented faithfully in the port (same behavior
spec, same messages where known) rather than copied from the backup:

- **Package dates control** (migration `drizzle/0007_package_dates.sql`,
  embedded like the rest): `businesses.package_start` / `package_end`
  (Pakistan calendar dates, YYYY-MM-DD). Super Admin sets them via
  `adminUpdateBusiness` (`packageStartDate`/`packageEndDate`) or
  `updateSettings` (Super Admin only); the admin panel has a First day /
  Last day + "Set dates" / "Clear dates" control per business, and the
  shop's Plan tab shows a Starts row (plus Ends and the window status).
  Outside the window the shop can view but cannot change anything that
  `assertUsable` already gates (orders, stock-in, due collections,
  supplier payments, expenses, products, etc.), with a starts/expired
  notice in the banner and on the Plan tab. Empty start clears the whole
  window; end before start is rejected; owners setting their own dates
  are refused ("Package dates are set by Super Admin." / "Super Admin
  only"). Renew (+30 days), suspend and reactivate work alongside.
  Scope note: the reference standalone v2 implementation gated only
  money/stock actions (orders, stock-in, payments, expenses); this port
  gates through the existing `assertUsable`, which is the stricter
  view-only reading — catalog edits are also blocked while the window
  is closed. Staff/profile/settings actions were already ungated for
  suspension/expiry and stay ungated, as before.
- **Navy palette 2** (`client/src/theme.css`, Muhammad's own
  palette, 2026-11-09, supersedes the bold and vibrant palette):
  very dark navy `#0b132b` background with `#1c2541` navy panels
  (`#253052` lighter tint) and `#3a506b` steel-blue borders; pale
  cream `#f4d6b0` text, muted `#cbb493`; light blue `#5bc0de`
  buttons/tabs with dark navy `#0b132b` text, `#3aa5cf` on hover;
  orange `#d97706` highlight (cart total, "Step 2" and plan
  "Current" badges) — also the only amber, used for genuine
  warnings (dues, expiry, low stock, not-started/expired banners,
  product-limit notices), so no second amber exists; steel blue
  `#3a506b` secondary accent ("Step 1" pill, white text). Plus
  Jakarta Sans display font, Inter body, 8px radius. Dark mode uses
  a deeper navy `#070f22` with the same light-blue / orange
  accents (hand-adapted). Red `#f87171` stays for suspended/errors.
  No bold green, burnt orange, yellow, mauve or peach token — and no
  old navy, cyan, indigo, emerald, blue or gold token — remains in
  the theme or the built CSS. Design source: Muhammad's navy
  palette image; see `NAVY-PALETTE-2-NOTES.md`.
- **Receipt** (`ReceiptModal` in `client/src/App.tsx`, 2026-11-09):
  the printed receipt now shows the signed-in business's own name
  large and bold at the top (from `state.settings.businessName` —
  never hard-coded), then a contact line with whatever the business
  profile has on file (phone / email / address, blanks omitted),
  then Order #, date + time, customer (or "Walk-in customer"),
  items with qty × rate and line totals (with product pictures),
  subtotal, discount if any, TOTAL, payment method with the cash /
  online / card split, paid, change, balance due if credit, RETURNED
  marking if returned, and a "Thank you for shopping with us!"
  footer. Printing is receipt-only: the print stylesheet hides the
  rest of the app, prints black-on-white (no dark backgrounds),
  centred at 80mm so it fits a thermal roll and A4. On screen the
  receipt keeps the navy palette. See `NAVY-PALETTE-2-NOTES.md`
  (incl. the virtual-printer print check).

## What changed, and why

| Original (space daemon) | This port | Why |
|---|---|---|
| `@hatch/space-sdk` server runtime | `server/src/space-shim.ts` (minimal `defineAction`/`Ctx`/`z`) | The daemon (stdin/stdout worker, `app.db`, daemon blob dir) does not exist on Vercel. The shim keeps action definitions byte-identical in behavior; only the import line in `actions.ts` changed (plus one type-only annotation: `ensureSeed(_db: Db)` so the project's own `tsc` passes — the original had a dead `Db` alias). |
| bun:sqlite file `app.db` | libSQL via `drizzle-orm/libsql` (`server/src/store.ts`) | Same SQLite dialect as the schema and migrations, so no SQL or schema changes. HTTP-based, so it works from serverless functions. A local SQLite file would be ephemeral on Vercel (the exact lesson from the first v2 deployment). |
| Daemon blob directory | Vercel Blob (private store) with a filesystem fallback (`BLOB_DIR`) for local runs | Product pictures need durable storage. Reads always use `useCache:false` — the Blob SDK caches private GETs by default, which previously served stale shop data for minutes. |
| Daemon `/actions` endpoint | `api/actions.ts` (Vercel Node function) | Dispatch mirrors the daemon: slugified action names (lowercase, separators → `_`), request validated (`invalid args: …`), handler errors surfaced, response validated (`response validation failed: …`), all as HTTP 200 `{error}` envelopes; unknown action → `action '…' is not defined`. Only transport failures (wrong method, bad JSON, DB down) use non-200 statuses. |
| Daemon `./blobs/<key>` serving | `api/blobs.ts` + `vercel.json` rewrites | `getUrl` still returns `./blobs/<base64url(key)>`; the route decodes, validates with the daemon's key rules, and streams bytes with the stored content type. Keys embed timestamps, so immutable caching is safe. |
| SDK `buildClient()` | `client/build.mjs` (own Bun.build, same config) | The SDK builder refuses to run outside the builder interface. Config replicated exactly (entry `client/index.html` → `client/dist`, minify, `NODE_ENV=production`, asset/chunk/entry naming, tailwind plugin, bare-asset-URL re-prefixing). PDF.js assets skipped — this client does not reference them. |
| SDK client (`createActionClient`, `spaceQueryClient`, `SafeAreaTopScrim`, `fileToBase64`) | `client/src/space-client.tsx` | Same POST `{action, args, actionCallId}` to `./actions`, same `{data}`/`{error}` handling, same retry policy (permanent statuses never retried; 5xx/429 retryable), same 30 s staleTime, no refetch on focus, same scrim rendering and chunked base64. Host-only behaviors (space-availability latching, viewer-auth replay, host theme postMessage) have no Vercel equivalent and are absent; theming already follows the browser's `prefers-color-scheme` without a host. |
| drizzle `migrate()` | Embedded migration runner (`server/src/migrations.ts` + `store.ts`) | `drizzle/meta/` holds only `_journal.json`, no snapshots, so the stock migrator cannot run. The runner splits the same `--> statement-breakpoint` files, applies one transaction per migration, tracks names in `__migrations`, and tolerates concurrent cold starts. Re-run `bun run embed:migrations` after any migration change. |

## File layout

```text
client/src/            App.tsx, api.ts, main.tsx, theme.css (unchanged except imports)
client/src/space-client.tsx   browser-side SDK replacement (new)
client/build.mjs       own client build (replaces SDK buildClient)
client/dist/           built client (regenerated by build:client)
server/src/actions.ts  all 43 actions (39 original + forgot-password x2 + wipe x1 + restock x1; import line + 1 type annotation only)
server/src/schema.ts   drizzle schema (unchanged)
server/src/space-shim.ts      server-side SDK replacement (new)
server/src/store.ts    libSQL + migrations + blob backends + Ctx (new)
server/src/migrations.ts      embedded drizzle SQL (generated, do not edit)
api/actions.ts         Vercel function: /actions (new)
api/blobs.ts           Vercel function: /blobs/<token> (new)
vercel.json            rewrites /actions → /api/actions, /blobs/:key → /api/blobs
scripts/embed-migrations.mjs  regenerates server/src/migrations.ts
scripts/prepare-vercel.mjs    builds client + assembles dist-vercel/
scripts/verify-local.ts       full local end-to-end verification (271 assertions)
dist-vercel/           deploy directory (regenerated; this is what ships)
```

`dist-vercel/` contains ONLY: `index.html`, `assets/`, `api/`, `server/`,
`package.json`, `bun.lock`, `vercel.json`. No `client/src`, no `drizzle/*.sql`,
no database, no blobs, no `space.json`.

## Deploy (do NOT run without the user's approval cards)

Prerequisites (deploy-time requirements, **not created by this task**):

1. **Turso account + database** (libSQL). Free tier is enough for testing;
   confirm the current allowance at deploy time. Create one database and note
   its URL and auth token.
2. **Private Vercel Blob store** connected to the Vercel project (supplies
   `BLOB_STORE_ID` + `VERCEL_OIDC_TOKEN`, or set `BLOB_READ_WRITE_TOKEN`).
   Metered; confirm the Hobby allowance at deploy time.
3. A Vercel project (the existing `pos-software` project can be reused, or a
   new one — the user's call at deploy time).

Steps:

1. `bun run prepare:vercel` (rebuilds the client, assembles `dist-vercel/`).
2. Deploy the **contents of `dist-vercel/`** as the project root
   (`vercel --prod` from inside it, or push to the project's repo).
3. Project settings: framework preset **none** (static + functions); **no
   build command** (the client is prebuilt); no output-directory override
   (root serves `index.html`). If the link is for partner testing, turn
   **Vercel SSO protection OFF** (it login-walls the site by default).
4. Environment variables on the project:
   - `TURSO_DATABASE_URL` — e.g. `https://<db>-<org>.turso.io`
   - `TURSO_AUTH_TOKEN` — Turso auth token for that database
   - Blob store connected (see above); no variable to set by hand when the
     store is connected through the Vercel dashboard.
5. First visit runs the 6 migrations automatically (fresh empty database),
   then shows the one-time Super Admin setup card. **Muhammad must do that
   setup himself with his own password** — nobody else may create it.

Costs: Vercel Hobby (free tier) + Turso free tier + Blob metered usage are
enough for partner testing; exact free allowances were not verified and must
be confirmed in the dashboards at deploy time. No other service is required.

## Local verification (done, all passing)

```sh
bun run typecheck    # client + server tsconfigs, clean
bun run verify:local # 271 assertions, fresh file: DB + filesystem blobs
bun run prepare:vercel
```

`verify:local` covers, end to end with hand-checked math: Super Admin setup
(incl. second-setup refusal), business signup, username checks, product add
with picture (serve via the blob route, mime/size refusals, replace, remove),
supplier/customer/stock-in, credit/split/cash guards (no-customer credit
refused, over-stock refused, split-mismatch refused), cash order with change
(200 total, 250 paid, 50 change), credit order (300), split order (80 cash +
120 online = 200), due collection (100), supplier payment (400), expense
(50), return, then full `getState` assertions — stock A 16 / B 8, supplier
purchased 1500 / paid 900 / due 600, customer due 200, money cashReceived
180 / onlineReceived 120 / cashPaidOut 900 / cashInHand −770 / bank 120 /
creditGiven 200 / creditTaken 600, P&L gross 500 / cogs 300 / gross profit
200 / net profit 150 / returns 200, inventory sell 2000 / cost 1200, today
sales 500 / orders 2 — plus staff lifecycle (cashier POS-only, self-disable/
self-delete refused, disable → login refused, delete → session invalid),
admin panel (overview, suspend/reactivate, expiry enforcement via
`updateSettings`, extendDays renewal, record payment → plan basic / 1999,
password reset, soft-delete), profile/settings guards, logout, wrong
password, and API-route parity (slug-insensitive dispatch, error envelopes,
405 on GET, blob 404) — plus package dates (future window blocks orders /
stock-in / collections / supplier payments while view still works,
today-covering window allows, ended window blocks, single-day today
window allows, end-before-start and bad dates rejected, owner
date-setting refused, window clear restores selling, suspend still works
alongside) and theme tokens (navy palette 2 — very dark navy / light blue / orange, no bold green, orange, yellow, mauve or peach, no old navy, cyan, indigo, emerald, blue or gold left)
— plus forgot password (no-key request errors with no code row, generic
message for unknown usernames, Super Admin / no-email errors, wrong /
cross-user / used / expired codes refused, 5 wrong attempts void the code,
stubbed-fetch send path: email to the business address, new request
invalidates the old code, emailed code resets the password, 6th request in
an hour refused) and wipe-all-data (non-admin and wrong confirm phrase
refused with nothing deleted; wipe empties all 16 tables, deletes the
product picture blob, `getSetupStatus` false, admin session dead, Super
Admin setup works again) — plus restock (500 → +500 → 1000 with the
stock-in row checked: no supplier, notes "Restock", cost 0, paid 0;
low-stock lift 3 → +10 → 13; supplier restock +100 @ 60 cash 2000:
purchased 7500, paid 2900, due 4600; refusals for 0 / negative /
fractional quantity, missing product, missing supplier, hidden product,
and cashier restock refused). Full detail in `FORGOT-PASSWORD-WIPE-NOTES.md` and `RESTOCK-NOTES.md`.

## What could NOT be preserved exactly — read before deploying

1. **Package dates + dark-blue theme are re-implementations, not copies.**
   Both post-date the Version 1 backup, so they were built from the
   2026-10-31 build's behavior spec (see "Added afterwards" above) and
   the standalone v2 reference, not from artifact source. They are
   locally verified (see below), but the exact wording/layout of the
   real artifact build's admin control could differ in details; the
   live deployed copy is where that gets compared.
2. **Action count: 39, not 40.** The brief lists 40 names; the backup's
   `actions.ts` defines 39 `defineAction` entries (the brief's own name list
   also counts 39 unique names — `adminGetOverview` appears twice, once as
   `admngetoverview`). Nothing is missing from the code; the brief's count
   was off by one.
3. **Two original behaviors preserved verbatim, worth knowing:**
   - A product row is inserted *before* its picture is saved, so a refused
     picture (bad mime, oversize) still leaves the product at stock 0.
   - `adminDeleteBusiness` does not delete anything — it only suspends the
     business. The owner can still log in afterwards but cannot use the shop.

## Still needs live verification after deployment (not done here)

- Turso connectivity from the Vercel function (cold start runs migrations;
  watch the first-visit latency — `getState` issues ~15 sequential Turso HTTP
  round trips, estimated 1–3 s, under the 10 s Hobby function limit, but
  unmeasured on a real deployment).
- Vercel Blob put/getUrl/serve round trip for a product picture (the Blob
  code path is the same SDK usage as the previously live-verified standalone
  deployment, but this port's Blob integration itself has only been verified
  against the filesystem backend locally).
- Vercel function bundling of `@libsql/client` (the https/Turso path is pure
  JS and loads no native binding, but the bundler's handling of the package's
  optional native module is unproven until the first deploy; if bundling
  chokes on it, the fallback is to externalize or alias the native entry).
- Browser UI click-through (API + static serving verified locally; partner
  use is the UI test, as before) — including clicking a package-date save
  in the live Super Admin panel and eyeballing the light professional tech theme on a
  phone screen (theme verified from tokens + built CSS, not a screenshot).
- Super Admin setup on the live link by Muhammad himself, and the partner
  signing up for his OWN business (never Muhammad's logins).
- Exact 23:59/00:00 PKT window edges: verified by date-string logic at
  day granularity locally, not by a faked clock (see the verify section
  note); the 10th–15th example from the artifact build was not re-run
  live.
