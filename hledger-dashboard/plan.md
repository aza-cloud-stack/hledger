Plan: Modern React UI for hledger

 Context

 The default hledger-web UI is functional but dated. We're building a modern React frontend that talks
 to hledger-web's JSON API, hosted on the same Ubuntu server behind nginx. The user is in India,
 tracking personal expenses in INR (₹), accessing from multiple devices via Tailscale.

 Tech Stack

 - Vite + React + TypeScript
 - shadcn/ui + Tailwind CSS (dark/light mode)
 - TanStack Query (server state)
 - React Router v6 (3 routes)
 - Recharts (dashboard charts)
 - nginx (reverse proxy — React static + API on single port)

 Project Location

 /Users/richardpaul/rich/hledger-dashboard/ (standalone, separate from hledger repo)

 Architecture

 Browser → nginx :8080
             ├── /          → React SPA (static files)
             └── /api/*     → proxy to hledger-web :5000

 hledger-web API Endpoints Used
 ┌─────────────────────────────┬────────┬─────────────────────────────────────┐
 │          Endpoint           │ Method │               Purpose               │
 ├─────────────────────────────┼────────┼─────────────────────────────────────┤
 │ /accounts                   │ GET    │ Account balances (dashboard)        │
 ├─────────────────────────────┼────────┼─────────────────────────────────────┤
 │ /accountnames               │ GET    │ Account autocomplete (expense form) │
 ├─────────────────────────────┼────────┼─────────────────────────────────────┤
 │ /transactions               │ GET    │ All transactions (list, dashboard)  │
 ├─────────────────────────────┼────────┼─────────────────────────────────────┤
 │ /accounttransactions/{name} │ GET    │ Per-account register                │
 ├─────────────────────────────┼────────┼─────────────────────────────────────┤
 │ /add                        │ PUT    │ Add new transaction (JSON body)     │
 └─────────────────────────────┴────────┴─────────────────────────────────────┘
 Routes
 ┌───────────────┬──────────────────┬─────────────────────────────────────────────┐
 │     Path      │       Page       │                   Purpose                   │
 ├───────────────┼──────────────────┼─────────────────────────────────────────────┤
 │ /add          │ ExpenseForm      │ Quick expense entry (default, mobile-first) │
 ├───────────────┼──────────────────┼─────────────────────────────────────────────┤
 │ /dashboard    │ DashboardPage    │ Charts + summary cards                      │
 ├───────────────┼──────────────────┼─────────────────────────────────────────────┤
 │ /transactions │ TransactionsPage │ Filterable transaction list                 │
 └───────────────┴──────────────────┴─────────────────────────────────────────────┘
 Project Structure

 hledger-dashboard/
 ├── vite.config.ts
 ├── tailwind.config.ts
 ├── components.json                    # shadcn/ui
 ├── nginx/
 │   └── hledger-dashboard.conf
 ├── src/
 │   ├── main.tsx
 │   ├── App.tsx                        # Router + QueryClient + AppShell
 │   ├── index.css
 │   ├── lib/utils.ts                   # shadcn cn()
 │   ├── api/
 │   │   ├── types.ts                   # TS interfaces matching hledger JSON
 │   │   ├── client.ts                  # fetchApi() wrapper → /api prefix
 │   │   ├── accounts.ts
 │   │   ├── transactions.ts
 │   │   └── add.ts                     # PUT /add
 │   ├── hooks/
 │   │   ├── use-accounts.ts            # useAccounts, useExpenseAccounts, usePaymentAccounts
 │   │   ├── use-transactions.ts        # useTransactions, useRecentTransactions
 │   │   ├── use-add-transaction.ts     # useMutation + cache invalidation
 │   │   └── use-dashboard-data.ts      # Compute monthly aggregations from raw data
 │   ├── utils/
 │   │   ├── amount.ts                  # decimalToNumber, sumAmounts, postingAmount
 │   │   ├── currency.ts                # formatINR() — Intl.NumberFormat en-IN
 │   │   ├── date.ts                    # Date helpers
 │   │   └── transaction-builder.ts     # Build full Transaction JSON for PUT /add
 │   └── components/
 │       ├── ui/                        # shadcn/ui (auto-generated)
 │       ├── layout/
 │       │   ├── app-shell.tsx          # Sidebar (desktop) + bottom nav (mobile)
 │       │   ├── mobile-nav.tsx         # Fixed bottom tabs: Add, Dashboard, Transactions
 │       │   └── theme-toggle.tsx       # Dark/light
 │       ├── expense-form/
 │       │   ├── expense-form.tsx       # Main form: date, desc, amount, category, payment, status
 │       │   ├── account-combobox.tsx   # Fuzzy autocomplete for expenses:* accounts
 │       │   ├── payment-select.tsx     # Dropdown for assets:/liabilities: accounts
 │       │   ├── status-toggle.tsx      # Unmarked/Pending/Cleared
 │       │   └── recent-transactions.tsx
 │       ├── dashboard/
 │       │   ├── dashboard-page.tsx     # Grid layout
 │       │   ├── summary-cards.tsx      # Income, Expenses, Savings, Savings Rate
 │       │   ├── income-expense-chart.tsx  # Recharts BarChart (12 months)
 │       │   ├── expense-breakdown.tsx  # Recharts donut (this month categories)
 │       │   ├── net-worth-chart.tsx    # Recharts AreaChart (trend)
 │       │   └── top-categories.tsx     # Sorted category list
 │       └── transactions/
 │           ├── transactions-page.tsx
 │           ├── transaction-table.tsx  # Sortable table
 │           ├── transaction-row.tsx    # Expandable row → posting details
 │           ├── transaction-filters.tsx # Search, date range, account filter
 │           └── posting-detail.tsx

 Implementation Phases

 Phase 1: Bootstrap

 1. npm create vite@latest hledger-dashboard -- --template react-ts
 2. Install Tailwind, shadcn/ui, add ~18 UI components
 3. Install: @tanstack/react-query, react-router-dom, recharts, date-fns, lucide-react
 4. Configure vite.config.ts with dev proxy (/api → localhost:5000)

 Phase 2: API + Types + Utilities

 5. src/api/types.ts — TS interfaces from hledger JSON (Transaction, Posting, Amount, Account, etc.)
 6. src/api/client.ts — base fetch wrapper
 7. src/api/accounts.ts, transactions.ts, add.ts
 8. src/utils/amount.ts, currency.ts (INR formatting), date.ts, transaction-builder.ts

 Phase 3: Layout + Routing

 9. App.tsx — QueryClientProvider + BrowserRouter + Routes
 10. app-shell.tsx — responsive layout with sidebar/bottom nav
 11. mobile-nav.tsx + theme-toggle.tsx

 Phase 4: Quick Expense Entry

 12. Hooks: use-accounts.ts, use-transactions.ts, use-add-transaction.ts
 13. account-combobox.tsx — expense account autocomplete
 14. payment-select.tsx + status-toggle.tsx
 15. expense-form.tsx — assemble form, wire PUT mutation
 16. recent-transactions.tsx — 5 most recent below form

 Phase 5: Dashboard

 17. use-dashboard-data.ts — compute monthly income/expense, category breakdown, net worth
 18. summary-cards.tsx — 4 metric cards
 19. Chart components: income-expense-chart.tsx, expense-breakdown.tsx, net-worth-chart.tsx
 20. top-categories.tsx + dashboard-page.tsx

 Phase 6: Transaction List

 21. transaction-filters.tsx — search, date range, account filter
 22. transaction-table.tsx + transaction-row.tsx + posting-detail.tsx
 23. transactions-page.tsx

 Phase 7: Polish + Deploy

 24. Dark mode (Tailwind class strategy + localStorage)
 25. Loading skeletons, error states, toast notifications
 26. nginx/hledger-dashboard.conf — reverse proxy config
 27. Build (npm run build), deploy to server, enable nginx

 Critical Implementation Details

 Transaction Builder (for PUT /add)

 Must produce exact JSON matching hledger's FromJSON Transaction. Every field required:
 buildTransaction({ date, description, amount, expenseAccount, paymentAccount, status })
 → full Transaction JSON with two postings (expense + payment), all fields populated
 Reference: hledger-web/Hledger/Web/Handler/AddR.hs → parseCheckJsonBody

 INR Formatting

 new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" })
 // → "₹1,23,456.78" (Indian lakhs/crores grouping)

 Dashboard Data Computation

 - Income = postings where paccount.startsWith("income:"), negated (income is negative in hledger)
 - Expenses = postings where paccount.startsWith("expenses:")
 - Net worth = accounts["assets"].aibalance - accounts["liabilities"].aibalance

 nginx Config (single port)

 server {
     listen 8080;
     root /var/www/hledger-dashboard;
     location / { try_files $uri $uri/ /index.html; }
     location /api/ {
         rewrite ^/api/(.*) /$1 break;
         proxy_pass http://127.0.0.1:5000;
     }
 }

 Verification

 1. Dev: npm run dev → open localhost:5173, verify API proxy works
 2. Expense entry: Add a transaction via form, verify it appears in ~/finance/2026.journal
 3. Dashboard: Verify charts render with real data from journal
 4. Transactions: Verify list shows all entries, filters work
 5. Mobile: Test on phone browser via Tailscale IP
 6. Production: Build, deploy to nginx, verify http://<tailscale-ip>:8080

