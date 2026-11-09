# POS Software — Vercel deployment

Live: https://pos-software-muhammad-saifs-projects.vercel.app

This repository holds the deployed build of the POS Software app
(multi-business point of sale, stock, suppliers, customers, dues,
expenses, reports and a Super Admin panel). It is deployed on Vercel
from this repository; every push to `main` auto-deploys.

## What is in this build

- Full POS app: POS counter, products, stock-in, suppliers, customers,
  orders, reports / profit & loss, staff roles, Super Admin panel
  (plans, payments, renew / suspend, package dates, reset owner
  password, reset all data).
- Plans: trial (14 days), Basic Rs 1,999, Standard Rs 3,999,
  Pro Rs 6,999 / month. Super Admin sets plan and package dates.
- Forgot password: a 6-digit code is emailed to the business email on
  file (needs `RESEND_API_KEY` set on the Vercel project; without it
  the app answers "Email sending is not set up yet").
- **Restock (2026-11-09):** a "Restock" button on every active product
  row on the Products tab. Enter the quantity to add — it adds to the
  stock on hand, never replaces it (500 pcs + restock 500 = 1,000 pcs).
  A supplier is optional; with a supplier, buy price and amount paid
  are recorded and supplier dues update like a stock-in purchase.
  Every restock is recorded as a stock-in row, so inventory, dues and
  reports stay consistent. Low-stock products (stock at or below the
  product's threshold, default 5) show an amber "Low stock" pill.
  Owner-only: cashiers are POS-only and cannot restock.
- Theme: bold and vibrant — dark green `#314039`, orange `#E17439`,
  yellow `#F6CB4B`, light `#F1D9CD`, bonus `#9983A8`.

## Technical notes

- React client + server actions, data in Turso (libSQL, Mumbai),
  product pictures in Vercel Blob. Env vars on the Vercel project:
  `TURSO_DATABASE_URL`, `TURSO_AUTH_TOKEN`, and optionally
  `RESEND_API_KEY` (+ `RESEND_FROM`).
- The client bundle is split into 7 part files under `assets/`
  (`index.html` reassembles them in the browser) and the server
  functions are prebundled CommonJS under `api/` — this keeps every
  file under Vercel's upload size limits.
- Source of truth for the build lives in the port workspace; this
  repository is the deployable snapshot only.
