# RASK – Full Product Documentation

**Version:** 1.0  
**Purpose:** Complete reference of all features and functionality for marketing and internal use.  
**Audience:** Management, marketing, and stakeholders.

---

## 1. Overview

**RASK** (Reseller Hub) is a web application for resellers and small businesses to manage inventory, sales, expenses, bank transactions, and marketplace integrations in one place. It supports multiple subscription plans (Free, Basic, Premium, Ultimate, and Beta Tester for testing), onboarding tasks, and a separate admin panel for app operators.

**Core value:** Track inventory, report sales, record expenses, connect marketplaces (eBay, Etsy, Shopify, Grailed, Depop, Poshmark, Mercari, Whatnot), sync orders and products where APIs exist, import via CSV where they do not as secondary option, connect bank accounts via Plaid, run P&L and tax-related reports, and reconcile sales to expenses.

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

**What it does:** Lets users create an account, sign in, recover their password, and manage their profile so only they can access their data.

- **Root (`/`):** Sends the visitor to the login page if they are not logged in, or to the dashboard if they are. This is the app’s entry point.
- **Registration:** Lets new users sign up with email and password. Admins can turn registration off in App Settings so no new accounts can be created.
- **Login:** Lets users sign in with either **email or username** and password. After success they are taken to the dashboard.
- **Logout:** Signs the user out and clears their session so the next visit requires login again.
- **Password reset:** Lets users request a reset link by email. They receive a link, set a new password, and can then log in again. The flow uses custom reset request plus Django’s reset/confirm pages.
- **Account settings:** Lets users view and update their profile (e.g. billing email). Plan is shown here; actual plan changes are done by admin/support, not by the user.

---

## 4. Dashboard

**What it does:** Gives a single overview of the business so the user can see key numbers and jump to the main areas of the app.

- **Dashboard home:** The first screen after login. It shows high-level metrics (e.g. total inventory count, sales summary, expenses, recent activity) and quick links to Inventory, Sales, Expenses, Reports, Integrations, and Finance. The user can see “where things stand” without opening each section.
- **Dashboard export:** Exports the same dashboard data (or a summary of it) as a CSV file so the user can use it in spreadsheets or share it.

---

## 5. Inventory

**What it does:** Lets the user keep a list of every item they have for sale—what they bought, how much they paid, how many they have, and who it belongs to (themselves or a consignor). This is the source for COGS and stock when reporting sales.

- **List:** Shows all inventory items in one place with title, SKU (if used), purchase price, purchase date, quantity, status (e.g. available/sold), consignor (if any), and extra fields (e.g. brand, color, size). User can search or filter to find items quickly.
- **Create:** Adds one new item. User enters title, optional SKU, purchase price, purchase date, quantity, status, and can link a consignor. Optional extra data (e.g. Poshmark-style brand, color, size) can be stored for reporting.
- **Edit:** Changes any field of an existing item (price, quantity, status, consignor, etc.).
- **Delete:** Removes an item from inventory permanently.
- **Bulk edit:** Lets the user select many items at once and change the same field(s) for all of them (e.g. update status or consignor for 50 items in one go).
- **Bulk delete:** Lets the user select many items and delete them in one action.

When a sale is recorded and linked to an inventory item, the app can decrement quantity; when quantity reaches zero, the item can be marked sold. Consignor linking is used later in the **Consignment Commission** report to calculate what is owed to each consignor.

---

## 6. Sales (Report Sale)

**What it does:** Records every sale the user makes—which item sold, when, for how much, and on which platform. This drives revenue in P&L, platform reports, and reconciliation. Sales can be entered by hand or brought in from marketplaces (sync or CSV).

- **List:** Shows all sales with filters (e.g. date range, platform). User can see history, totals, and open a sale to view or edit.
- **Create:** Records a new sale: user picks the inventory item, date, quantity sold, sale price, platform (eBay, Poshmark, etc.), and optional notes. Optional marketplace fields (order ID, brand, color, size, cost, tax, etc.) can be filled for better reporting. If the sale came from a marketplace, an external ID can be stored to avoid duplicates when syncing or re-importing.
- **Edit:** Updates any detail of an existing sale (price, date, platform, notes, etc.).
- **Delete:** Removes a sale from the records (e.g. if it was entered by mistake or cancelled).
- **Details (JSON):** Returns full sale details in a structured format for use in modals or other UI (e.g. “View details” popup) without loading a full page.
- **Bulk edit:** Lets the user select multiple sales and change the same field(s) for all (e.g. change platform or add a note to many rows at once).

Sales with **external_id** and **platform** allow the app to recognize the same order when it is synced or imported again, so the same sale is not created twice.

---

## 7. Expenses (Transaction Expense)

**What it does:** Records every business expense—shipping, supplies, fees, gas, etc.—so the user can track spending and see net profit (revenue minus COGS and expenses). Expenses can also be matched to sales in reconciliation.

- **List:** Shows all expenses with date, payee, amount, memo. User can scan spending and use filters if needed.
- **Create:** Adds a new expense: date, payee (e.g. USPS, Amazon), amount, memo, optional GL account (for categorization), and a flag for whether it is **sourcing** (cost of acquiring inventory) or not. Sourcing vs non-sourcing helps with reporting and COGS.
- **Edit:** Updates any field of an existing expense.
- **Delete:** Removes an expense from the records.
- **Bulk edit:** Lets the user select multiple expenses and change the same field(s) for all (e.g. set GL account or sourcing flag for many rows).

GL account and sourcing flag are used in reports and in understanding where money is going (e.g. shipping vs. cost of goods).

---

## 8. Vendors and Consignment

**What it does:** Keeps a list of people or companies the user does business with—suppliers, shipping partners, or **consignors** (people who give items to sell on commission). Consignors get a commission rate; the app uses this to compute how much to pay them in the Consignment Commission report.

- **List:** Shows all vendors with name, contact info, and whether they are marked as a consignor. User can see who they work with and open one to edit or delete.
- **Create:** Adds a new vendor: name, contact (email, phone, address), notes, and optionally marks them as a **consignor** and sets a **commission rate** (e.g. 50%). This rate is used when calculating commission owed on sales linked to that consignor.
- **Edit:** Updates vendor details or commission rate.
- **Delete:** Removes the vendor. (If inventory or sales are linked, the app may handle that per business rules.)

Inventory items and sales can be linked to a consignor. The **Consignment Commission** report then shows how much is owed to each consignor based on their sales and commission rate, so the user can pay them correctly.

---

## 9. Transaction Details

**What it does:** Shows all money movement in one place—sales (money in), expenses (money out), and (if connected) bank transactions—so the user can see a clear timeline of income and spending, and export it.

- **Transaction Details page:** Combines sales, expenses, and bank transactions (when Plaid is connected) in a single view, typically by date or category. The user can see “what came in and what went out” without jumping between Sales and Expenses. Visiting this page also completes the “View Transaction Details” task in Get Started.
- **Export:** Exports this combined transaction list to CSV so the user can use it in Excel, for accountants, or for their own records.

---

## 10. Transaction Matching (Reconciliation)

**What it does:** Lets the user tie specific expenses to specific sales over a date range (e.g. “this $5 shipping expense goes with this $50 sale”). That way they can see which costs belong to which sale and what is still unmatched—useful for understanding true profit per order or for bookkeeping.

- **Home:** Lists all reconciliation sessions (each session is a date range the user chose). User can open an existing session or create a new one.
- **New session:** Creates a reconciliation session: user picks a from-date and to-date and can add a note (e.g. “January 2025”). The session holds all matches for that period.
- **Session detail:** Shows the sales and expenses in that date range, which ones are already matched (sale ↔ expense, full or partial amount), and how much is still unmatched on each sale or expense.
- **Add match:** Links one sale and one expense and assigns an amount (full or partial). For example: “$5 of this $8 shipping expense matches this $50 Poshmark sale.” The app records the match and updates remaining amounts.
- **Delete match:** Removes a match so that sale and expense are no longer linked; amounts become unmatched again.
- **Close session:** Marks the session as closed so the user knows they are done reconciling that period (no lock—it is a status only).

This helps resellers see which expenses tie to which sales and track unmatched amounts for follow-up or reporting.

---

## 11. Reseller Reports

**What it does:** Gives the user ready-made reports for profit, inventory value, platform performance, taxes, and consignment payouts. All report pages support date filters where relevant. Some reports (e.g. Tax Calculator) may be limited by plan (e.g. Premium/Ultimate).

### 11.1 Profit & Loss (P&L)

Shows how the business performed over a date range: **revenue** (from sales), **COGS** (cost of goods sold, from inventory cost), **gross profit** (revenue − COGS), **expenses** (from the expense list), and **net profit** (gross profit − expenses). The user can see if they are making money and where the money goes. **Export** downloads this summary as a CSV (e.g. for an accountant or records).

### 11.2 Inventory Valuation

Shows the **total value of current inventory**—e.g. sum of (quantity × cost) for all items still in stock—so the user knows how much capital is tied up in unsold goods. Optional search/filter by item. **Export** downloads the valuation list as CSV.

### 11.3 Sales by Platform

Breaks down sales **by selling platform** (eBay, Poshmark, Depop, etc.) for the chosen date range. The user can see which platforms bring in the most revenue and compare performance. **Export** downloads the platform breakdown as CSV.

### 11.4 Per Item Analysis

Shows **performance per inventory item**: how many units sold, revenue, cost, and profit per item. Helps the user see which products are profitable and which are not. **Export** downloads the per-item data as CSV.

### 11.5 Tax Calculator

Provides **tax-related views or estimates** (e.g. income subject to tax, rough tax estimate) based on the user’s sales and expenses. Useful for planning and filing. Access may be limited to certain plans (e.g. Premium/Ultimate).

### 11.6 Schedule C Generator

Organizes the user’s revenue and expense data in a structure that aligns with **IRS Schedule C** (US sole-proprietor business tax form), so they or their accountant can transfer numbers more easily to the actual form.

### 11.7 Accrual Accounting

Shows revenue and expenses in an **accrual** view (when earned/incurred) rather than only when cash moved. Helps users who need accrual-based books or reports for lenders, investors, or tax.

### 11.8 Consignment Commission

Calculates **how much commission is owed to each consignor** based on sales linked to them and their commission rate. The user can see a list of payouts (e.g. “Vendor A: $120, Vendor B: $85”) so they can pay consignors correctly.

---

## 12. Get Started (Onboarding)

**What it does:** Guides new users through the main actions so they set up inventory, record a sale, add an expense, connect a marketplace or bank, and open key reports. A checklist shows progress and reduces the chance they miss important steps.

- **Get Started page:** Displays a list of tasks (e.g. connect a marketplace, add your first inventory item, report your first sale, add your first expense, create a reconciliation session, view the P&L report, connect your bank account, view Transaction Details, view Per Item Analysis). Each task can be marked done or still to do.
- **Toggle task:** Lets the user manually mark a task as done or undo it. Many tasks are also **auto-marked** when the user does the action (e.g. first sale created → “Report your first sale” is marked done; first expense → “Add your first expense” is marked done).

Tasks are stored per user so progress is saved. This gives new resellers a clear path to get value from the app quickly.

---

## 13. Marketplace Integrations

**What it does:** Lets the user connect their selling accounts (eBay, Etsy, Shopify, Grailed, Depop, Poshmark, Mercari, Whatnot) so sales and sometimes inventory can be pulled into RASK automatically (sync) or so they can import orders via CSV for that account. Reduces manual data entry and keeps books in sync with marketplaces. The **Integrations** list is the central place: user adds an account (Connect or CSV), syncs it, or removes it. How many connections and which marketplaces can use “Connect” depend on the user’s plan.

### 13.1 General Integration Actions

- **List:** Shows all connected marketplaces (and any “CSV-only” placeholders) with status (e.g. connected, last sync). User can see what is linked and open one to sync or delete.
- **Connect marketplace:** User chooses a provider (e.g. Poshmark) and a store/account name; the app creates a row for that account. Then the user either completes **Connect** (OAuth or login flow) for that provider or uses **CSV import** for orders. So “connect” here means “add a marketplace account to my list.”
- **Delete integration:** Removes that marketplace account from the user’s list. Sync and Connect for that account stop; the user can re-add later if needed.
- **Sync:** Triggers a refresh for that account: the app fetches orders (and, where supported, inventory) from the provider’s API or stored auth and creates/updates sales (and inventory) in RASK. Actual behavior is provider-specific (eBay pulls orders + listings, Shopify orders + products, etc.).

### 13.2 eBay

**What it does:** Connects the user’s eBay account via **OAuth** (secure sign-in with eBay). After connection, **Sync** pulls orders and listing/inventory data from eBay’s API into RASK so sales and inventory stay up to date without manual entry. Available on Basic and above (not Free). Plan limits how many eBay accounts can be connected (e.g. 2 on Premium, 5 on Ultimate). OAuth start redirects the user to eBay to consent; callback receives the code and exchanges it for tokens; sync runs on demand for that account.

### 13.3 Etsy

**What it does:** Same idea as eBay: **OAuth** connection to the user’s Etsy shop, then **Sync** pulls orders (and shop data as supported) into RASK. Reduces manual entry for Etsy sellers. Same plan logic as eBay (Basic+; connection limits by plan).

### 13.4 Shopify

**What it does:** Connects the user’s Shopify store via **OAuth**. User enters their shop domain (e.g. mystore.myshopify.com); they are sent to Shopify to authorize the app; the app stores the access token. **Sync** then fetches orders and products from the Shopify Admin API so sales and inventory in RASK match the store. Uses a single public app (credentials in .env); any store can connect without an App Store listing. Same plan rules as eBay/Etsy.

### 13.5 Grailed

**What it does:** For Grailed there is no public OAuth, so **Connect** uses the user’s email and password (and area/store name). The app logs in via a browser-automation approach, stores session tokens/cookies, and then **Sync** fetches sales from Grailed’s API so orders appear as sales in RASK. **Connect** is only available on **Ultimate** or **Beta Tester**; Free/Basic/Premium users cannot use Connect and must use **CSV import** for Grailed orders.

### 13.6 Depop

**What it does:** **Connect** is a two-step flow: (1) user requests a magic link to their Depop email, (2) user pastes that link back into RASK; the app exchanges it for a session and saves the account. **Sync** then pulls orders from Depop (when the endpoint is wired) into RASK. **Connect** only on **Ultimate** or **Beta Tester**.

### 13.7 Poshmark

**What it does:** **Connect** captures the user’s Poshmark credentials (or session) and stores them; **Sync** uses that to fetch sales so they appear in RASK. When the provider’s login/sales endpoints are fully wired, sync will populate sales automatically. **Connect** only on **Ultimate** or **Beta Tester**. Users on lower plans can still use **CSV import** for Poshmark orders.

### 13.8 Mercari

**What it does:** Same pattern as Poshmark: **Connect** (Ultimate/Beta only) stores auth; **Sync** pulls orders into RASK when supported. **CSV import** is available for all plans as an alternative.

### 13.9 Whatnot

**What it does:** Same pattern: **Connect** (Ultimate/Beta only) and **Sync** to bring Whatnot sales into RASK; **CSV import** available for other plans.

---

## 14. CSV Imports

**What it does:** Lets the user bulk-import **orders (sales)**, **inventory**, or **expenses** from CSV files when they don’t use Connect/sync or when they prefer to upload exports from a marketplace or bank. Saves time vs. entering rows one by one. **Duplicate-file detection:** The app hashes each uploaded file; if the same file is uploaded again, it is rejected and the user is told that file was already imported—this avoids double-counting sales or expenses.

### 14.1 Orders (Sales) CSV

User selects a marketplace (provider) and an account (acct_id), then uploads a CSV of orders (e.g. downloaded from Poshmark, Depop, eBay, Etsy, Mercari, Whatnot). The app detects the header row and maps columns per marketplace (or uses a generic format), then creates or updates sales in RASK. Each file’s hash is stored so the same file cannot be imported twice.

### 14.2 Inventory CSV

User uploads a CSV of inventory items. The app supports Poshmark-style or standard columns (e.g. title, purchase_price, purchase_date, quantity, status) and creates inventory records. File hash is stored to prevent re-importing the same file.

### 14.3 Expenses CSV

User uploads a CSV with required columns (date, payee, amount, memo, gl_account, is_sourcing). The app creates expense records. File hash is stored so the same file cannot be imported again.

---

## 15. Finance (Bank Connections and Plaid)

**What it does:** Lets the user connect their **bank account(s)** via **Plaid** so real bank transactions appear in RASK. They can then view transactions, refresh them (sync), and **allocate** transactions to sales or expenses (e.g. “this bank debit is the shipping expense for that order”). Bank data is used in **Transaction Details** and in reporting so the user has a full picture of cash flow. Plan may limit how many bank connections a user can add.

- **Bank list:** Shows all connected bank accounts (institution name, last sync, etc.). User can open one to see transactions, sync, or delete.
- **Transactions for a bank:** Shows the list of transactions for that connection (date, description, amount, etc.). User can see what hit the account and optionally allocate items to sales/expenses.
- **Delete connection:** Removes the bank connection; transactions for that account are no longer fetched. User can reconnect later.
- **Sync:** Asks Plaid for the latest transactions and updates the list so the user sees recent activity without re-linking the account.
- **Plaid link token:** Back-end provides a link token so the front-end can open **Plaid Link** (Plaid’s UI where the user picks their bank and signs in). No credentials are entered into RASK; Plaid handles auth.
- **Plaid exchange:** After the user completes Plaid Link, the front-end sends the one-time public token here; the app exchanges it with Plaid for an access token and stores the connection so future syncs can pull transactions.
- **Plaid webhook:** Plaid sends webhooks (e.g. when an account has new data or is disconnected). The app receives them and can update connection status or trigger syncs as needed.
- **Allocation create:** User links a bank transaction to a sale or an expense (e.g. “this $50 deposit is from this Poshmark sale” or “this $8 debit is this shipping expense”). Creates an allocation record so reports and Transaction Details can show the link.
- **Allocation delete:** Removes an allocation so that bank transaction is no longer tied to that sale or expense.

---

## 16. Admin Panel

**What it does:** Separate area for **app operators** (not resellers). Admins log in at `/admin-panel/` and can control who can sign up, who is an admin, and approve or decline admin registration requests. Regular users do not see this; it is for running the product.

- **Login / Logout / Password reset:** Same idea as main app but for admin accounts; keeps admin access secure.
- **Register (admin):** Lets someone request an admin account (e.g. username, email, password). Can be disabled in App Settings so no new admin sign-ups are possible.
- **Dashboard:** Overview of the app: e.g. total user count, number of pending admin approval requests. Quick glance at system health and pending work.
- **Settings:** Turn **user registration** on or off (whether new resellers can sign up) and **admin registration** on or off (whether new admin requests are accepted). Single place to lock or open sign-ups.
- **Users list:** Lists all regular (reseller) users. Admin can see who is in the system and open a user if needed.
- **Delete user:** Permanently removes a reseller account and their data (use with care).
- **Admins list:** Lists all admin users so the operator can see who has admin access.
- **Delete admin:** Removes an admin’s access so they can no longer log in to the admin panel.
- **Approval requests:** Lists pending admin sign-up requests (people who asked to become an admin and are waiting for approval).
- **Approve / Decline request:** Approves a request (the person becomes an admin and can log in) or declines it (the request is closed and they do not get access). Ensures only trusted people get admin rights.

---

## 17. Security and Data

**What it does:** Protects user data and prevents common attacks so resellers can trust the app with sales, expenses, and bank links.

- **Encryption:** Sensitive data (marketplace tokens, stored passwords, Plaid/bank tokens) is encrypted at rest. If the database is exposed, these values are not readable without the app’s key.
- **Sessions:** Login and OAuth flows use Django sessions so the server knows who is logged in and can safely complete “redirect to eBay/Shopify and back” without exposing state to the client.
- **CSRF:** All forms and state-changing requests require a CSRF token so requests must come from the app’s own pages, reducing cross-site request forgery.
- **File hashes:** For CSV imports, the app stores only a hash of each file (not the file itself). That way it can block duplicate uploads without keeping raw file content.

---

## 18. Summary Table of All Features

| Area | What it does (short) |
|------|----------------------|
| **Auth** | Register, sign in with email or username, sign out, reset password, update account/profile. |
| **Dashboard** | One-screen overview of key metrics and quick links; export that view as CSV. |
| **Inventory** | List, add, edit, delete items (single or bulk); link consignors and extra fields; quantity/status for COGS and reporting. |
| **Sales** | List, add, edit, delete sales (single or bulk); link to inventory and platform; optional marketplace fields and external_id for sync/CSV. |
| **Expenses** | List, add, edit, delete expenses (single or bulk); GL account and sourcing flag for reporting. |
| **Vendors** | List, add, edit, delete vendors; mark consignors and set commission rate for commission report. |
| **Transaction Details** | Single view of sales + expenses + bank transactions; export to CSV. |
| **Reconciliation** | Create sessions by date range; link sales to expenses (full/partial); view unmatched amounts; close session. |
| **Reports** | P&L, inventory valuation, sales by platform, per item, tax calculator, Schedule C, accrual, consignment commission; CSV exports where listed. |
| **Get Started** | Onboarding checklist; mark tasks done (manual or auto when user does the action). |
| **Integrations** | Add/remove marketplace accounts; Connect (OAuth or login) and Sync per provider (eBay, Etsy, Shopify, Grailed, Depop, Poshmark, Mercari, Whatnot); plan limits. |
| **CSV** | Import orders (per provider/account), inventory, or expenses from CSV; block duplicate files by hash. |
| **Finance** | Connect banks via Plaid; list transactions; sync; allocate transactions to sales/expenses; delete connection. |
| **Admin** | Operator login; enable/disable user and admin registration; list/delete users and admins; approve/decline admin requests. |

---

## 19. Plans and Feature Access (Recap)

- **Free:** Up to 3 marketplace connections; orders via CSV only (no OAuth); one account per marketplace.
- **Basic:** Up to 8 connections; OAuth for eBay, Etsy, Shopify; one account per provider.
- **Premium:** Up to 16 connections; up to 2 OAuth per provider; up to 2 accounts per non-API marketplace; Connect for Grailed/Depop/Poshmark/Mercari/Whatnot **disabled** (CSV only).
- **Ultimate:** Unlimited connections; up to 5 OAuth per provider; up to 5 accounts per marketplace; **Connect enabled** for Grailed, Depop, Poshmark, Mercari, Whatnot.
- **Beta Tester:** Same Connect access as Ultimate for testing (no payment).

---

*End of RASK documentation. For technical implementation details, see codebase and NON_API_MARKETPLACES_FILES_LIST.md where applicable.*
