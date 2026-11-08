# POS Software — deploy build

Deployable build of the POS Software app (multi-business point-of-sale SaaS
in PKR), ported from the Muse artifact app. This directory is the project
root on Vercel: prebuilt static client + serverless API functions, no build
command, framework preset none.

## Theme

Light professional tech theme (Stitch redesign 2026-11-08, "POS Software —
Professional Tech Redesign", Stitch project
`projects/9598143607219987445`): clean white `#ffffff` background, slate
surfaces `#f8fafc` / `#f1f5f9`, hairline `#e2e8f0` borders, dark slate
`#0f172a` text, indigo `#4f46e5` buttons/tabs with white text (`#4338ca`
hover), Inter font, 8px radius. Amber `#f59e0b` only for genuine warnings
(dues, expiry, low stock); red `#dc2626` for suspended/errors. Supersedes
the charcoal + emerald theme and the dark blue + gold theme.

## App

- 42 server actions: POS orders (cash / credit / split Cash+Online+Card+
  Credit), products with pictures, suppliers + stock-in, customers,
  payables/receivables, expenses, returns, P&L reports, staff roles
  (cashier POS-only), subscription plans (Trial / Basic Rs 1,999 /
  Standard Rs 3,999 / Pro Rs 6,999), Super Admin panel (businesses, plans,
  payments, renew, suspend, package dates).
- Forgot password: 6-digit code sent to the business email on file.
  Requires the `RESEND_API_KEY` environment variable (Resend) to send mail;
  without it the action answers "Email sending is not set up yet".
  Super Admin accounts have no email on file and cannot use it.
- Super Admin "Reset all data": type `RESET ALL DATA` to permanently wipe
  every business, user, product, order and record (including the Super
  Admin account itself) for a fresh start; Super Admin setup must be
  redone afterwards.

## Data

- Turso (libSQL, Mumbai) via `TURSO_DATABASE_URL` + `TURSO_AUTH_TOKEN`
  (Production env vars on the Vercel project).
- Product pictures in the private Vercel Blob store linked to the project.

## Files

- `index.html` + `assets/` — prebuilt client (bundle split into 7 parts,
  loaded in order by an inline loader; CSS with the theme tokens).
- `api/actions.js`, `api/blobs.js` — prebundled CommonJS serverless
  functions (`/actions`, `/blobs/:key` via `vercel.json` rewrites).
- `package.json`, `bun.lock`, `vercel.json`.

Source of this build: `~/workspace/pos-artifact-vercel/` (port, local
verification 211/211 assertions, typecheck clean).
