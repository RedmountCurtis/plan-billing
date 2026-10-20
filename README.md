# Redmount Plan Billing

401(k) plan fee invoicing for Redmount Capital Partners. Schwab accounts stay on Blueleaf; this covers the plans.

Single-file app (`index.html`) on GitHub Pages, Supabase backend, same pattern as the rest of the Redmount suite.

## What it does

- **Plans** — each 401(k) plan with sponsor/contact, billing method (invoice sponsor vs. deduct from plan assets), frequency, arrears/advance, valuation method, proration, payment terms, and its fee schedule.
- **Fee schedules** — tiered (graduated), breakpoint (whole balance at one rate), flat bps, or flat dollar; plus optional base fee, per-participant fee, annual minimum/maximum, and discount. Live fee check on the plan screen.
- **New invoice** — pick plan and period (calendar presets or custom dates, optional mid-period service start), enter values, exclusions (loans, SDBA, etc.), participants, and credits/adjustments. Live preview of the invoice as you type.
- **Billing run** — every active plan on a schedule in one table; type values or paste straight from Excel (`CODE, value, excluded, participants`), review fees, create all drafts at once. Plans already invoiced for the period are skipped.
- **Review workflow** — Draft → Reviewed → Approved → Sent → Paid, with timestamps and who did each step. Figures lock at approval (enforced in the database). Corrections = Void and re-issue. One live invoice per plan per period.
- **Output** — "Invoice" (amount due, due date, remit instructions) or "Fee Statement" (deduct-from-plan wording) with full calc detail: billable assets, each tier, min/max, adjustments, effective bps. DRAFT watermark until approved. Print / Save PDF (Letter, one page), print a selected batch, CSV export, JSON backup.

## Setup

### 1. Database (Supabase)
Supabase → Redmount project (`pdympbpyavilstjsmjtm`) → SQL Editor → paste all of `schema.sql` → Run.
Creates `billing_plans`, `billing_invoices`, `billing_settings`, with RLS restricted to
`hollowell@redmountcapital.com` and `grigorian@redmountcapital.com` (edit the list at the top of the file to add anyone).

### 2. GitHub
Create repo `redmountcurtis/plan-billing` → upload `index.html` → Settings → Pages → deploy from `main` / root.
Live at `https://redmountcurtis.github.io/plan-billing/`. Supabase URL and key are already set (same as the Terminal).

### 3. Redmount Terminal
Replace the Terminal's `index.html` with `terminal/index.html` (adds the Plan Billing tile next to Fiduciary Monitor; nothing else changed).

## Login
Uses the Terminal's session (`localStorage['redmount_session']`) and the same raw-fetch auth — no Supabase client library.
Sign in on either page and you're signed in on both; signing out of either signs out of both. Token refreshes are written back to the shared session.

## Test mode
Add `?local` to the URL (`…/plan-billing/?local`) to use the app with data saved only in that browser. Nothing touches Supabase.

## Calculation notes
- Fees are annual rates converted to the period: **Periodic** = annual ÷ periods per year (prorated by days for partial periods); **Actual/365** or **Actual/Actual** = days in period ÷ year.
- Order: asset fee + base + per-participant → annual min/max (prorated) → discount → adjustments.
- Average valuation averages the entered month-end values (blanks ignored). "Average daily balance" takes the ADB figure from the recordkeeper.
- Each invoice stores a snapshot of the plan terms, schedule, inputs, and calc — editing a plan later never changes an issued invoice.
