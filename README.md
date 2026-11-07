# POS Software — Vercel deployment

Multi-business point of sale and business management SaaS (PKR): POS
punching, products with buy/sell prices, suppliers, customers, stock-in,
udhaar (credit) ledgers, split payments, expenses, returns, P&L reports,
staff roles, and a Super Admin panel (plans, payments, renew/suspend,
package dates).

This repository holds the **deployable build** served by Vercel at
https://pos-software-muhammad-saifs-projects.vercel.app — not the editable
source. It is deployed as-is: framework preset none, no build command, no
output-directory override. Environment variables on the Vercel project:
`TURSO_DATABASE_URL` and `TURSO_AUTH_TOKEN` (Turso libSQL database), plus
the connected private Vercel Blob store for product pictures.

## Design

Dark blue + gold + white (redesigned 2026-11-08): dark blue majority,
gold for hover and tab effects, white borders.

## How this build is produced

The editable source lives outside this repo (the artifact-app port). Each
release is regenerated and pushed here:

```sh
bun run prepare:vercel            # rebuild client, assemble dist-vercel/
bun ./scripts/chunk-bundle-for-upload.mjs   # split the 342KB client bundle
bun ./scripts/bundle-functions-for-vercel.mjs  # prebundle api/*.js (CommonJS)
# then push the contents of dist-vercel/ to this repo (branch main)
```

Notes:

- `assets/index-*.partN` are byte slices of the single client bundle;
  `index.html` fetches them in order, concatenates them byte-for-byte and
  imports the result. The bundle code itself is unmodified.
- `api/actions.js` / `api/blobs.js` are prebundled CommonJS serverless
  functions (externals `@libsql/client`, `drizzle-orm`, `@vercel/blob`,
  `zod` are installed by Vercel from `package.json`).
- The first visit runs the database migrations automatically, then shows
  the one-time Super Admin setup card. The Super Admin account is created
  by the owner himself with his own password — never by anyone else.
