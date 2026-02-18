# RASK – Full Product Documentation

**Version:** 1.0  
**Purpose:** Complete reference of all features and functionality for marketing and internal use.  
**Audience:** Management, marketing, and stakeholders.

---

## 1. Overview

**RASK** (Reseller Hub) is a web application for resellers and small businesses to manage inventory, sales, expenses, bank transactions, and marketplace integrations in one place. It supports multiple subscription plans (Free, Basic, Premium, Ultimate, and Beta Tester), onboarding tasks, and a separate admin panel for app operators.

**Core value:** Track inventory, report sales, record expenses, connect marketplaces (eBay, Etsy, Shopify, Grailed, Depop, Poshmark, Mercari, Whatnot), sync orders and products where APIs exist, import via CSV where they do not, connect bank accounts via Plaid, run P&L and tax-related reports, and reconcile sales to expenses.

---

## 2. User Roles and Subscription Plans

### 2.1 Plans

| Plan | Description | Limits (summary) |
|------|-------------|------------------|
| **Free** | Entry tier | Up to 3 marketplace connections; OAuth (eBay, Etsy, Shopify) not available—CSV import only; one account per marketplace. |
| **Basic** | Mid tier | Up to 8 marketplace connections; OAuth available; one account per marketplace. |
| **Premium** | Higher tier | Up to 16 marketplace connections; up to 2 OAuth connections per provider (eBay, Etsy, Shopify); up to 2 accounts per non-API marketplace (Grailed, Depop, Poshmark, Mercari, Whatnot). |
| **Ultimate** | Top tier | Unlimited marketplace connections; up to 5 OAuth connections per provider; up to 5 accounts per marketplace; **only plan that can use Connect for Grailed, Depop, Poshmark, Mercari, Whatnot** (non-API marketplaces). |
| **Beta Tester** | Testing | Same Connect access as Ultimate for all marketplaces (including non-API) for testing; no payment. |

Plan is stored per user in **User Profile** (account email, billing email, plan). Feature gating (e.g. OAuth, number of connections, which marketplaces can “Connect”) is based on this plan.

### 2.2 User Types

- **Regular user (reseller):** Signs up, has a plan, uses dashboard, inventory, sales, expenses, reports, integrations, finance, and get-started tasks.
- **Admin:** Separate login (`/admin-panel/`). Can manage users, other admins, approval requests, app settings (e.g. registration on/off). Does not use the main Django admin at `/admin/` for app business logic.

---

## 3. Authentication and Account Management

- **Root:** `/` redirects to login (or dashboard if already logged in).
- **Registration:** Sign up at `/accounts/register/` (can be disabled via App Settings).
- **Login:** Email or username at `/accounts/login/`.
- **Logout:** POST to `/accounts/logout/`.
- **Password reset:** Custom flow at `/accounts/password-reset/`; Django’s password reset templates (V3) for reset link and confirmation.
- **Account settings:** `/settings/account/` – user can update profile (e.g. billing email, plan display); plan changes are typically done by admin/support.

---

## 4. Dashboard

- **Dashboard home:** `/dashboards/` – main landing after login. Shows high-level metrics (e.g. inventory count, sales summary, expenses, recent activity) and quick links.
- **Dashboard export:** `/dashboards/export/` – export dashboard data as CSV.

---

## 5. Inventory

- **List:** `/add-inventory/` – view all inventory items (title, SKU, purchase price, purchase date, quantity, status, consignor, extra data).
- **Create:** `/add-inventory/new/` – add a single item (title, SKU optional, purchase price, purchase date, quantity, status, consignor).
- **Edit:** `/add-inventory/<id>/edit/` – edit an item.
- **Delete:** `/add-inventory/<id>/delete/` – delete an item.
- **Bulk edit:** `/add-inventory/bulk-edit/` – select multiple items and edit in bulk.
- **Bulk delete:** `/add-inventory/bulk-delete/` – select multiple items and delete.

Inventory supports **consignor** (vendor) linking and **extra_data** (e.g. Poshmark brand, color, size). Quantity can be decremented when a sale is created (sold status when quantity reaches zero).

---

## 6. Sales (Report Sale)

- **List:** `/report-sale/` – list all sales with filters (date range, platform, etc.).
- **Create:** `/report-sale/new/` – record a sale (item, date, quantity, sale price, platform, notes, optional marketplace fields).
- **Edit:** `/report-sale/<id>/edit/` – edit a sale.
- **Delete:** `/report-sale/<id>/delete/` – delete a sale.
- **Details (JSON):** `/report-sale/<id>/details/` – sale details for modals or API-like use.
- **Bulk edit:** `/report-sale/bulk-edit/` – bulk edit selected sales.

Sales can have **external_id** and **platform** for deduplication (e.g. from marketplace sync). Optional fields: listing_date, order_id, department, category, brand, color, size, cost_price, sales_tax, etc., for marketplace-specific data.

---

## 7. Expenses (Transaction Expense)

- **List:** `/add-transaction/` – list expenses.
- **Create:** `/add-transaction/new/` – add expense (date, payee, amount, memo, gl_account, is_sourcing).
- **Edit:** `/add-transaction/<id>/edit/` – edit expense.
- **Delete:** `/add-transaction/<id>/delete/` – delete expense.
- **Bulk edit:** `/add-transaction/bulk-edit/` – bulk edit expenses.

Expenses support **GL account** and **sourcing** flag for reporting and categorization.

---

## 8. Vendors and Consignment

- **List:** `/vendors/` – list vendors.
- **Create:** `/vendors/new/` – add vendor (name, is_consignor, commission_rate, email, phone, address, notes).
- **Edit:** `/vendors/<id>/edit/` – edit vendor.
- **Delete:** `/vendors/<id>/delete/` – delete vendor.

**Consignors** are vendors marked with `is_consignor` and optional **commission rate**. Used in inventory (consignor link), sales (consignor), and **Consignment Commission** report. Commission is used for payout and reporting.

---

## 9. Transaction Details

- **Page:** `/transaction-details/` – view combining sales, expenses, and (if applicable) bank transactions in a unified view (e.g. by date or category).
- **Export:** `/transaction-details/export/` – export transaction details to CSV.

Used for a clear view of money in/out and to complete the “View Transaction Details” onboarding task.

---

## 10. Transaction Matching (Reconciliation)

Reconciliation matches **sales** to **expenses** (e.g. sale revenue vs. shipping cost) over a date range.

- **Home:** `/transaction-matching/` – list reconciliation sessions.
- **New session:** `/transaction-matching/new/` – create session (from_date, to_date, note).
- **Session detail:** `/transaction-matching/<session_id>/` – view matches and remaining amounts.
- **Add match:** `/transaction-matching/<session_id>/add-match/` – link a sale and an expense (full or partial amount).
- **Delete match:** `/transaction-matching/<session_id>/delete-match/<match_id>/` – remove a match.
- **Close session:** `/transaction-matching/<session_id>/close/` – mark session closed.

Helps resellers see which expenses tie to which sales and track unmatched amounts.

---

## 11. Reseller Reports

All report pages support date filters where relevant. Access to some reports (e.g. Tax Calculator) may be gated by plan (e.g. Premium/Ultimate).

### 11.1 Profit & Loss (P&L)

- **Report:** `/reseller-reports/` – revenue, COGS, gross profit, expenses, net profit for a date range.
- **Export:** `/reseller-reports/export/` – P&L CSV.

### 11.2 Inventory Valuation

- **Report:** `/reseller-reports/inventory/` – value of inventory (e.g. by cost or quantity).
- **Export:** `/reseller-reports/inventory/export/`.

### 11.3 Sales by Platform

- **Report:** `/reseller-reports/platforms/` – sales broken down by platform (eBay, Poshmark, etc.).
- **Export:** `/reseller-reports/platforms/export/`.

### 11.4 Per Item Analysis

- **Report:** `/reseller-reports/per-item/` – performance per inventory item (e.g. sales, profit per item).
- **Export:** `/reseller-reports/per-item/export/`.

### 11.5 Tax Calculator

- **Report:** `/reseller-reports/tax-calculator/` – tax-related view/estimates (plan-gated).

### 11.6 Schedule C Generator

- **Report:** `/reseller-reports/schedule-c/` – data structured for Schedule C (US tax).

### 11.7 Accrual Accounting

- **Report:** `/reseller-reports/accrual/` – accrual-based view of revenue/expenses.

### 11.8 Consignment Commission

- **Report:** `/reseller-reports/consignment-commission/` – commission owed to consignors based on sales and commission rates.

---

## 12. Get Started (Onboarding)

- **Page:** `/get-started/` – checklist of tasks to complete (e.g. connect marketplace, add inventory, report first sale, add expense, reconciliation, view P&L, connect bank, view transaction details, per item analysis).
- **Toggle task:** `/get-started/toggle/<key>/` – mark a task done/undone.

Tasks are stored per user. Completing relevant actions (e.g. first sale, first expense) auto-marks the corresponding task.

---

## 13. Marketplace Integrations

Central place: **Integrations** list at `/integration/`. User can **Connect** (OAuth or credentials), **Sync** (pull orders/products), use **CSV** import, or **Remove** an account. Limits (how many connections, which marketplaces can use Connect) depend on plan.

### 13.1 General Integration Actions

- **List:** `/integration/` – all connected marketplaces and their status.
- **Connect marketplace:** `/integration/connect/` – choose provider and store name to add a new row (then Connect or CSV per provider).
- **Delete integration:** `/integration/<id>/delete/` – remove a marketplace account.
- **Sync (generic):** `/integration/<id>/sync/` – trigger sync for that account (actual sync is provider-specific).

### 13.2 eBay

- **OAuth start:** `/integration/ebay/start/<acct_id>/` – redirect to eBay consent.
- **Callback:** `/integration/ebay/callback/` – receive code, exchange for tokens, save. An alternate callback URL `/ebay/oauth/callback/` exists for local/testing compatibility.
- **Sync orders:** `/integration/ebay/sync/<acct_id>/` – fetch orders and inventory from eBay API.

Available on Basic and above (not Free). Plan limits: e.g. 2 connections on Premium, 5 on Ultimate.

### 13.3 Etsy

- **OAuth start:** `/integration/etsy/start/<acct_id>/`.
- **Callback:** `/integration/etsy/callback/`.
- **Sync orders:** `/integration/etsy/sync/<acct_id>/`.

Same plan logic as eBay.

### 13.4 Shopify

- **OAuth start:** `/integration/shopify/start/` or `/integration/shopify/start/<acct_id>/` – user enters shop domain, redirect to Shopify OAuth.
- **Callback:** `/integration/shopify/callback/` – exchange code for access token, save shop and token.
- **Sync orders:** `/integration/shopify/sync/<acct_id>/` – fetch orders and products from Shopify Admin API.

Uses public app credentials (Client ID/Secret in .env). Same plan rules as eBay/Etsy.

### 13.5 Grailed

- **Connect start:** `/integration/grailed/start/` or `/integration/grailed/start/<acct_id>/` – show login form (email, password, area, store name).
- **Connect process:** `/integration/grailed/process/` – submit credentials; app logs in via cloudscraper/Playwright, stores tokens and cookies.
- **Sync orders:** `/integration/grailed/sync/<acct_id>/` – fetch sales from Grailed API using stored auth.

**Connect** only on **Ultimate** or **Beta Tester**. Free/Basic/Premium see Connect disabled; they can use CSV import.

### 13.6 Depop

- **Connect start:** `/integration/depop/start/` or `/integration/depop/start/<acct_id>/` – two-step: (1) request magic link to email, (2) paste magic link to complete.
- **Connect process:** `/integration/depop/process/` – step 1 sends link; step 2 exchanges link for session (cookies/token), saves account.
- **Sync orders:** `/integration/depop/sync/<acct_id>/` – fetch orders (endpoint to be wired when available; currently placeholder).

**Connect** only on **Ultimate** or **Beta Tester**.

### 13.7 Poshmark

- **Connect start:** `/integration/poshmark/start/` or `/integration/poshmark/start/<acct_id>/`.
- **Connect process:** `/integration/poshmark/process/` – credentials stored; login/sales endpoints to be filled when provided.
- **Sync orders:** `/integration/poshmark/sync/<acct_id>/`.

**Connect** only on **Ultimate** or **Beta Tester**. CSV import supported.

### 13.8 Mercari

- **Connect start:** `/integration/mercari/start/` or `/integration/mercari/start/<acct_id>/`.
- **Connect process:** `/integration/mercari/process/`.
- **Sync orders:** `/integration/mercari/sync/<acct_id>/`.

**Connect** only on **Ultimate** or **Beta Tester**. CSV import supported.

### 13.9 Whatnot

- **Connect start:** `/integration/whatnot/start/` or `/integration/whatnot/start/<acct_id>/`.
- **Connect process:** `/integration/whatnot/process/`.
- **Sync orders:** `/integration/whatnot/sync/<acct_id>/`.

**Connect** only on **Ultimate** or **Beta Tester**. CSV import supported.

---

## 14. CSV Imports

All imports support **duplicate-file detection**: same file (SHA256 hash) cannot be imported twice; user sees a message that the CSV was already uploaded.

### 14.1 Orders (Sales) CSV

- **URL:** `/imports/orders/<provider>/<acct_id>/` – e.g. provider = poshmark, depop, ebay, etc.; acct_id = marketplace account ID.
- **Flow:** Upload CSV; app detects header row and maps columns per marketplace; creates/updates sales; stores file hash to block re-import of same file.

Supports marketplace-specific column mapping (e.g. Poshmark, Depop, eBay, Etsy, Mercari, Whatnot) and generic format.

### 14.2 Inventory CSV

- **URL:** `/imports/inventory/`.
- **Flow:** Upload CSV; Poshmark-style or standard format (title, purchase_price, purchase_date, quantity, status); creates inventory items; file hash stored to prevent duplicate import.

### 14.3 Expenses CSV

- **URL:** `/imports/expenses/`.
- **Flow:** Upload CSV with required headers (date, payee, amount, memo, gl_account, is_sourcing); creates expenses; file hash stored to prevent duplicate import.

---

## 15. Finance (Bank Connections and Plaid)

- **Bank list:** `/finance/` – list connected bank accounts.
- **Transactions for a bank:** `/finance/<id>/txns/` – view transactions for one connection.
- **Delete connection:** `/finance/<id>/delete/`.
- **Sync:** `/finance/<id>/sync/` – refresh transactions from Plaid.
- **Plaid link token:** `/finance/plaid/link-token/` – get link token to launch Plaid Link (front-end).
- **Plaid exchange:** `/finance/plaid/exchange/` – exchange public token for access token and store connection.
- **Plaid webhook:** `/finance/plaid/webhook/` – receive Plaid webhooks (e.g. updates).
- **Allocation create:** `/finance/alloc/create/<txn_id>/` – allocate a bank transaction to a sale or expense.
- **Allocation delete:** `/finance/alloc/delete/<alloc_id>/` – remove an allocation.

Bank data is used in **Transaction Details** and reporting. Plan-based limits may apply (e.g. number of connections).

---

## 16. Admin Panel

Separate area for app operators. Base path: `/admin-panel/`.

- **Login:** `/admin-panel/login/`.
- **Register (admin):** `/admin-panel/register/` – can be disabled via App Settings.
- **Logout:** `/admin-panel/logout/`.
- **Password reset:** `/admin-panel/password-reset/`.
- **Dashboard:** `/admin-panel/` – overview (e.g. user count, pending requests).
- **Settings:** `/admin-panel/settings/` – e.g. enable/disable user registration, admin registration.
- **Users list:** `/admin-panel/users/` – list regular users.
- **Delete user:** `/admin-panel/users/<user_id>/delete/`.
- **Admins list:** `/admin-panel/admins/` – list admin users.
- **Delete admin:** `/admin-panel/admins/<admin_id>/delete/`.
- **Approval requests:** `/admin-panel/approval-requests/` – list pending admin sign-up requests.
- **Approve request:** `/admin-panel/approval-requests/<request_id>/approve/`.
- **Decline request:** `/admin-panel/approval-requests/<request_id>/decline/`.

---

## 17. Security and Data

- **Encryption:** Sensitive fields (e.g. marketplace tokens, passwords, bank tokens) are encrypted at rest using app-level encryption.
- **Sessions:** Django sessions for login and OAuth state (e.g. Shopify state, eBay state).
- **CSRF:** All forms and state-changing requests use CSRF protection.
- **File hashes:** CSV import hashes (orders, inventory, expenses) are stored to prevent duplicate uploads; no raw file content stored.

---

## 18. Summary Table of All Features

| Area | Features |
|------|----------|
| **Auth** | Register, login (email/username), logout, password reset, account settings |
| **Dashboard** | Dashboard home, dashboard CSV export |
| **Inventory** | List, create, edit, delete, bulk edit, bulk delete; consignor; extra_data |
| **Sales** | List, create, edit, delete, details, bulk edit; platform; external_id |
| **Expenses** | List, create, edit, delete, bulk edit; gl_account; is_sourcing |
| **Vendors** | List, create, edit, delete; consignor flag; commission rate |
| **Transaction Details** | View, CSV export |
| **Reconciliation** | Sessions: new, detail, add match, delete match, close |
| **Reports** | P&L, inventory valuation, sales by platform, per item, tax calculator, Schedule C, accrual, consignment commission; exports where listed |
| **Get Started** | Onboarding checklist, toggle task |
| **Integrations** | List, connect (add account), delete, sync; per-provider OAuth/Connect and sync (eBay, Etsy, Shopify, Grailed, Depop, Poshmark, Mercari, Whatnot); plan limits |
| **CSV** | Orders (per provider/account), inventory, expenses; duplicate-file blocking |
| **Finance** | Bank list, transactions, delete, sync, Plaid link/exchange/webhook, allocations |
| **Admin** | Login, register, logout, password reset, dashboard, settings, users, admins, approval requests (approve/decline) |

---

## 19. Plans and Feature Access (Recap)

- **Free:** 3 marketplaces; CSV only for orders; no OAuth; one per provider.
- **Basic:** 8 marketplaces; OAuth (eBay, Etsy, Shopify); one per provider.
- **Premium:** 16 marketplaces; 2 OAuth per provider; 2 accounts per non-API marketplace; Connect for non-API disabled.
- **Ultimate:** Unlimited; 5 OAuth per provider; 5 per non-API; **Connect enabled for Grailed, Depop, Poshmark, Mercari, Whatnot**.
- **Beta Tester:** Same Connect access as Ultimate for testing.

---

*End of RASK documentation. For technical implementation details, see codebase and NON_API_MARKETPLACES_FILES_LIST.md where applicable.*
