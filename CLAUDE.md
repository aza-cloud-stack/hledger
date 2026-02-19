# hledger Personal Finance Setup — Context

## Owner
- Location: India
- Goal: Track personal expenses using hledger with industry-standard double-entry accounting
- Access: Self-hosted hledger-web on Linux home server, accessible from all devices

## Project Status
- [ ] Finalize chart of accounts
- [ ] Create initial journal file with opening balances
- [ ] Install hledger + hledger-web on Linux home server
- [ ] Configure hledger-web for LAN/remote access
- [ ] Set up reverse proxy (nginx/caddy) with HTTPS
- [ ] Import historical bank data (CSV)
- [ ] Establish daily/weekly/monthly workflow

## Currency
- Primary: INR (₹)
- Numbering: Indian system (lakhs/crores) — `₹ 1,00,000.00`

## Chart of Accounts (India-specific)

```
assets
├── bank
│   ├── <bank>:savings        ; Primary salary account
│   ├── <bank>:savings        ; Secondary account
│   └── <bank>:fd             ; Fixed deposits
├── cash                      ; Cash in hand/wallet
├── investments
│   ├── mutualfunds           ; SIPs, lump sum
│   ├── stocks                ; Demat holdings
│   ├── ppf                   ; Public Provident Fund
│   ├── epf                   ; Employee PF
│   ├── nps                   ; National Pension
│   └── gold                  ; Physical/digital gold
├── receivables               ; Money others owe you
└── upi                       ; GPay/PhonePe balance (if any)

liabilities
├── creditcard
│   └── <bank>
├── loans
│   ├── homeloan
│   ├── carloan
│   └── personal
└── payables                  ; Money you owe others

income
├── salary
│   ├── basic
│   ├── hra
│   ├── special
│   └── bonus
├── freelance
├── interest                  ; FD/savings interest
├── dividends
├── rental
├── cashback                  ; CC/UPI cashback
└── gifts

expenses
├── housing
│   ├── rent
│   ├── maintenance           ; Society maintenance
│   └── utilities (electricity, water, gas)
├── food
│   ├── groceries
│   ├── dining                ; Restaurants
│   └── delivery              ; Swiggy/Zomato
├── transport
│   ├── fuel
│   ├── auto                  ; Auto/Ola/Uber
│   ├── metro
│   └── maintenance           ; Vehicle servicing
├── telecom (mobile, broadband)
├── shopping (clothing, electronics, household)
├── health (medical, insurance, pharmacy)
├── education
├── entertainment
│   ├── subscriptions         ; Netflix/Hotstar/Spotify
│   └── outings
├── personal (grooming, fitness)
├── tax
│   ├── incometax
│   ├── tds                   ; Tax Deducted at Source
│   └── gst                   ; If freelancing
├── insurance (life, health, vehicle)
├── gifts
├── charity                   ; Donations (80G eligible)
├── emi                       ; Loan EMI payments
└── miscellaneous

equity
└── opening                   ; Opening balances
```

## Hosting Plan
- Server: Linux home server
- App: hledger-web
- Bind: `0.0.0.0:5000`
- Journal path: TBD
- Reverse proxy: TBD (nginx or caddy for HTTPS + auth)
- Access: LAN + optional Tailscale/WireGuard for remote

## Workflow
- Daily: Log expenses via hledger-web (phone/laptop browser)
- Weekly: Review & categorize uncleared transactions
- Monthly: Reconcile bank statements, log salary/SIPs/EMIs
- Quarterly: Income statement, budget review
- Yearly: Balance sheet, tax prep (80C/80D), close books

## Key Commands
```bash
hledger bal expenses -M                    # Monthly expense breakdown
hledger bal assets                         # What you own
hledger is -M                              # Monthly income statement
hledger bs                                 # Net worth (balance sheet)
hledger bal expenses:food -W               # Weekly food spending
hledger reg creditcard                     # All credit card transactions
hledger bal expenses -p thismonth --depth 2  # This month summarized
```

## Transaction Patterns

### Salary
```journal
YYYY-MM-01 * Employer - Month Salary
    assets:bank:<bank>:savings      ₹75,000
    expenses:tax:tds                ₹15,000
    assets:investments:epf           ₹5,000
    income:salary                  ₹-95,000
```

### UPI/Cash/Card Expense
```journal
YYYY-MM-DD * Description
    expenses:<category>             ₹amount
    assets:bank:<bank>:savings               ; UPI/debit
    ; or: liabilities:creditcard:<bank>      ; credit card
    ; or: assets:cash                        ; cash
```

### SIP
```journal
YYYY-MM-DD * SIP - Fund Name
    assets:investments:mutualfunds  ₹5,000
    assets:bank:<bank>:savings
```

### Credit Card Bill
```journal
YYYY-MM-DD * CC Bill Payment
    liabilities:creditcard:<bank>   ₹amount
    assets:bank:<bank>:savings
```

---
*Last updated: 2026-02-19*
