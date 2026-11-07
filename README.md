# POS Software — Vercel deploy build

Deployable build of the actual POS Software artifact app (ported from the
Version 1 backup; see the port directory `~/workspace/pos-artifact-vercel/`).
This repository holds ONLY the built output — `index.html`, `assets/`,
`api/`, `package.json`, `bun.lock`, `vercel.json`. No source, no database,
no business data.

## This build (2026-11-08)

- **Theme:** dark premium charcoal + emerald, designed through Stitch
  (Stitch project "POS Software — Dark Premium Redesign",
  `projects/11904778197395997465`). Near-black charcoal `#0c1210`
  background, `#141b18` / `#1b2420` panels, vivid emerald `#10b981`
  buttons/tabs, warm off-white `#f4f1ea` text. Supersedes the dark blue +
  gold + white theme. See `STITCH-REDESIGN-NOTES.md` in the port directory.
- **42 server actions** (39 original + 3 added 2026-11-08):
  `requestPasswordReset`, `resetPasswordWithCode`, `adminResetAllData`.
- **Forgot password:** a 6-digit reset code is emailed to the business
  email on file. Sending uses the Resend API and requires the
  `RESEND_API_KEY` environment variable (plus optional `RESEND_FROM`) on
  the Vercel project. Until that key is set, "Forgot password?" answers
  "Email sending is not set up yet." No SMS/phone delivery. Super Admin
  accounts have no email on file, so they cannot use forgot-password.
- **Reset all data (Super Admin only):** the Super Admin panel has a
  "Reset all data — fresh start" section. It wipes every business, user,
  product, order and record (confirm by typing `RESET ALL DATA`). It
  also removes the Super Admin account itself, so Super Admin setup must
  be done again afterwards.
- **Environment variables on the Vercel project:** `TURSO_DATABASE_URL`,
  `TURSO_AUTH_TOKEN`, private Blob store connected, optional
  `RESEND_API_KEY` / `RESEND_FROM`.
- **Local verification before this build:** typecheck clean,
  `bun run verify:local` 208/208 assertions pass.

## Deploy notes

- Framework preset: none (static + functions). No build command, no
  output-directory override — the client is prebuilt.
- `vercel.json` rewrites `/actions` to `/api/actions` and
  `/blobs/:key` to `/api/blobs`.
- The client bundle is split into 7 part files (`assets/index-*.part0–6`)
  loaded in order by an inline loader in `index.html`; the server
  functions are prebundled CommonJS (`api/actions.js`, `api/blobs.js`).
