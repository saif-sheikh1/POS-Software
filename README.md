# POS Software — Vercel deployment

Live link: https://pos-software-muhammad-saifs-projects.vercel.app

## This build — bold and vibrant palette (2026-11-09)

Muhammad's own palette, applied to the POS app (styling only, no
logic changes):

- Background / dark surfaces: dark green `#314039` (replaces black/navy)
- Panels: `#3c4f45` / `#46584d`, borders `#5b6f62`
- Text: light peach `#f1d9cd` on dark surfaces
- Primary buttons: burnt orange `#e17439` with white text
- Highlight: yellow `#f6cb4b` (cart total, "Step 2" and plan "Current" badges)
- Secondary accent: mauve `#9983a8` ("Step 1" pill)
- Amber `#f59e0b` warnings only; red `#f87171` errors/suspended

Local verification: 221/221 assertions pass, typecheck clean.
See `BOLD-PALETTE-NOTES.md` in the source directory for the full
mapping and the one open flag (printed bonus hex `#9983A8` is mauve,
but the swatch in his image looks sage-green).

## What this is

The actual POS Software app (multi-business SaaS in PKR): POS
counter, products/stock, suppliers, customers, dues, expenses,
reports/P&L, staff roles, Super Admin panel (plans, payments, renew,
suspend, package dates, reset-all-data wipe). Trial 14 days, then
Basic Rs 1,999 / Standard Rs 3,999 / Pro Rs 6,999 per month.

## Data

Backed by a Turso (libSQL) database in Mumbai and a private Vercel
Blob store for product pictures. Sign up with your own business —
never use anyone else's login.
