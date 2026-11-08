# POS Software — Vercel deploy build

Deploy build of the actual POS Software artifact app (ported from the
Version 1 backup), built 2026-11-08.

## Theme

Dark navy (Stitch "Dark Navy Redesign", project
`projects/13654573335552685556`, 6 screens): deep navy `#0b1326`
background, navy panels `#131b2e` / `#171f33`, cyan accent `#06b6d4`
with dark teal-navy `#003640` button text, pale lavender-white
`#dae2fd` text, Plus Jakarta Sans display font, Inter body, 8px
radius. Amber is warnings-only; red is suspended/errors. Supersedes
the light white/indigo theme, the charcoal/emerald theme, and the
dark blue/gold theme.

## App

- 42 server actions (39 original + `requestPasswordReset`,
  `resetPasswordWithCode`, `adminResetAllData`).
- Multi-business SaaS in PKR: POS, products, stock-in, suppliers,
  customers, dues ledgers, expenses, P&L reports, staff roles,
  Super Admin panel (plans, payments, renew/suspend, package dates,
  password reset, **Reset all data** wipe).
- Forgot password: 6-digit code to the business email via Resend.
  Needs the `RESEND_API_KEY` env var on the Vercel project — without
  it the request answers "Email sending is not set up yet". Super
  Admin accounts have no email on file and cannot use it.
- Reset all data: Super Admin only, type exactly `RESET ALL DATA`.
  Deletes every business, user, product, order and record including
  the Super Admin account itself — Super Admin setup must be redone
  afterwards.

## Files

Prebuilt client (chunked bundle parts) + prebundled CommonJS server
functions (`api/actions.js`, `api/blobs.js`). No build step on Vercel.
Env vars required: `TURSO_DATABASE_URL`, `TURSO_AUTH_TOKEN`
(Turso libSQL, Mumbai), plus the connected private Vercel Blob store
for product pictures.

Full port notes: `~/workspace/pos-artifact-vercel/README.md`,
`STITCH-REDESIGN-NAVY-NOTES.md`, `FORGOT-PASSWORD-WIPE-NOTES.md`.
