# Libeara Product Knowledge Base

> Purpose: Onboarding reference and product understanding guide. Covers what exists, what's being built, and what's planned.
> Last updated: 2026-05-19

---

## 1. Platform Overview

### What is Libeara
Tokenization platform (incubated by SC Ventures / Standard Chartered) for regulated fund managers. Investors subscribe to and redeem from tokenized funds using fiat or stablecoin. MAS-licensed.

### Three Products

| Product | Description | Status |
|---------|-------------|--------|
| **Delta (Vault dApp)** | Legacy investor platform — wallet-connect onboarding | Live, users being migrated to LIP |
| **LIP (Libeara Investment Platform)** | New MAS-licensed platform — email signup | Active development |
| **Crane** | Separate SCB admin-only product | Separate — NEVER mix with Delta/LIP |

**Important:** Crane is a completely separate product. Never mix Crane content, flows, or documentation with Delta/LIP.

### Key Concept: FOP (Free of Payment)
The core design principle. Tokens move on-chain, cash settles off-chain via bank. The platform stitches both legs together with full status tracking. This is why sub/redemption flows are complex — they must track both blockchain state and off-chain payment state simultaneously.

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | 11 NestJS microservices on Azure AKS (ArgoCD blue-green deployment) |
| Smart Contracts | 3 per fund per chain: KYC Contract, Delta Token (ERC-20), Delta Manager V2 |
| Chains | Ethereum, Solana, Avalanche, Arbitrum (XRPL in progress) |
| On-chain signing | Fireblocks (MPC platform) |
| Admin auth | Azure AD B2C (OTP) |
| Email | SendGrid (45 system + 13 dynamic templates) |
| AML screening | Chainalysis (daily wallet re-scan) |
| Database | Cosmos DB (per-fund isolation) |
| KYC/KYB | Chekk |
| Banking | ACME (SCB/UOB integration) |
| On/Off-ramp | StraitsX (fiat ↔ stablecoin conversion) |
| Treasury | Wellington (fund treasury management) |

### Smart Contract Architecture (3 per fund per chain)
1. **KYC Contract** — maintains whitelisted investor wallet addresses
2. **Delta Token (ERC-20)** — token contract, handles minting and burning
3. **Delta Manager V2** — orchestrates subscription/redemption logic; consolidates Delta Manager + Fiat contracts into one

---

## 2. What Exists Now (Live)

### Wallet Management
- Investors connect wallets (MetaMask, Sui Wallet, CoinBase)
- Wallet whitelisting via KYC Contract
- Chainalysis AML screening on all wallets (daily re-scan)
- Admin can view all investor wallets and whitelist status

### Task Management
- Tasks auto-created when investor actions require admin review (bank account approval, subscription acceptance, etc.)
- FA/FM can view, assign, and complete tasks
- Linked to subscription/redemption orders and bank account requests

### Onboarding (LIP Phase 1)

**Account Types:** Individual, Corporate/Institutional

**Account Statuses:**
| Status | Meaning |
|--------|---------|
| PENDING | Signed up, not yet approved; limited to own account page |
| ACTIVE | Fully approved; full platform access |
| SUSPENDED | Can login, limited to account page only |
| INACTIVE | Cannot login (offboarded) |

**Individual Onboarding:**
1. Investor completes Basic Information
2. Ops creates KYC ID in Chekk portal
3. Ops sends email + KYC link to investor
4. After KYC review: Verified → Ops approves → ACTIVE; Not Verified → SUSPENDED with reason

**Corporate Onboarding:**
1. Investor completes Basic Corporate + Authorized Person info
2. Ops creates KYB IDV in Chekk portal
3. Ops sends email + KYB link
4. After Chekk verification → same approval flow as individual

**Key distinction:** For Individual, user status syncs with Main Account status. For Corporate/Institutional, Main Account is approved first → User status stays PENDING until separately approved.

### RBAC (Role-Based Access Control)

**Roles:**
- **Super Admin** — highest access, full platform control
- **Fund Admin (FA)** — operational actions per fund (accept/reject orders, resolve)
- **Fund Manager (FM)** — review/approval per fund
- **Custom roles** — created by Super Admin with granular permissions

**Permission structure:** Module (Red) → Category/Screen (Orange) → Feature (Green). Selecting a parent auto-selects all children. Permissions are additive when a user has multiple roles.

**Enforced at both UI and API layers:**
- Screen-level: controls menu visibility + direct URL access
- Feature-level: controls specific actions within accessible screens

**Member Management:**
- Add/Edit/Delete members; assign roles and fund access
- Change status Active ↔ Disabled (forces logout when disabling)
- Cannot edit email after creation; full activity log for audit

**Role Management:**
- Create/Edit/Duplicate/Delete roles
- Edit role forces logout of all affected users
- Delete only allowed if role not assigned to any users

### Bank Account Module

**Purpose:** Central place for clients to register and manage bank accounts for fiat transactions.

**Client Actions:**
- Add bank account: currency, beneficiary name, bank name, account number, SWIFT, correspondent bank, documents → submits for admin verification
- View all linked accounts with status
- Update account (triggers re-verification)
- Delete account (only if not linked to active investments or pending transactions; data kept for audit)

**Admin Actions:**
- View all bank accounts, approve/reject, request re-submission
- Add bank account on behalf of investor (Maker-Checker)
- Approve/reject triggers task in Task Management; full audit log

**Bank Account Statuses:**
| Status | Meaning |
|--------|---------|
| PENDING APPROVAL | Submitted by client, awaiting admin review |
| ACTION REQUIRED | Admin needs more info/documents from client |
| ACTIVE | Approved, usable for redemptions |
| INACTIVE | Rejected or failed verification |
| PENDING REMOVE | Removal requested for active account (needs admin approval) |
| REMOVED | Soft-deleted, kept in DB for audit |

**Status Transition Scenarios:**
- S1: Client submits → PENDING APPROVAL
- S2: Admin approves → ACTIVE
- S3: Admin needs more info → ACTION REQUIRED
- S4: Client resubmits → back to PENDING APPROVAL
- S5: Client doesn't respond → INACTIVE
- S6: Client requests remove (active) → PENDING REMOVE → REMOVED (admin approval needed); non-active → automatically REMOVED
- S7: Client edits active account → PENDING APPROVAL (re-verification required)
- S8: Admin approves edit → ACTIVE
- S9: Admin adds on behalf of client → PENDING APPROVAL → ACTIVE/INACTIVE

---

## 3. What's Being Built Now

### Subscription & Redemption Redesign (Delta Manager V2)
The biggest ongoing initiative. Redesigns the entire sub/redemption flow to properly support FOP. See Section 5 for full flow details.

**Why it's being redesigned:**
- Current flow doesn't properly handle the FOP model (on-chain tokens + off-chain cash)
- New Delta Manager V2 contract consolidates Delta Manager + Fiat contracts
- Adds Notification of Intent (NI) step before order submission
- Proper stablecoin locking and automatic refund logic

**Status:** PRD & Design complete. Smart contract tickets blocked pending SC team capacity.

### LIP Migration Script
Migrates existing Delta (Vault dApp) users to LIP. Handles user data, wallet associations, and transaction history.

### XRPL Integration
Adding XRP Ledger as a supported chain.

### Solana SC Audit Fixes
Addressing critical and medium findings from the Solana smart contract security audit.

### Token Management Module
Admin-facing interface to manage the full token lifecycle. See Section 6 for details. Status: Pending kickoff.

### ACME Integration
Banking integration with SCB/UOB for payment instructions and transaction confirmation. Required for the fiat leg of sub/redemption. Status: PRD complete, pending kickoff.

### Email Notifications
Comprehensive email notification system covering all investor and admin touchpoints. Status: PRD & Design phase.

### AI PR Review Bot
Automated code review bot for pull requests. Status: Testing.

### Datadog Phase 1
Observability and monitoring setup. Status: Pending kickoff.

---

## 4. What's Coming Next (Planned / Clarifying)

### 2FA Implementation
Two-factor authentication for the platform. Currently in clarification phase.

### Ensemble 2.0 (HKMA)
HKMA portal enhancements — 7 requirements identified, targeting June 2026.

### Ultra Bridge (Delta App)
Cross-chain bridge for Delta tokens. TDD draft exists with Plan A vs Plan B options — product decision pending.

### Libby / LibearaGPT (AI Initiative)
AI LLM assistant for the platform. Terms of Use draft in progress. Under product review.

### Crane Solana Expansion
Deploy CAMC funds (including Gold ETFs) on Solana chain. Target: end June 2026.

### Automation Testing
Full test automation suite. Planned but not yet started.

### Crane API Access
Programmatic API access for Crane platform clients.

### Parking Lot (No Active Timeline)
- LIP Brand Identity — visual identity refresh
- Fund Storefront (Static Pages) — public-facing fund information pages
- Statements of Holdings / Account Balance — investor portfolio statements

---

## 5. Core Flows: Subscription & Redemption (Redesigned)

> This describes the **target state** being built (Delta Manager V2). The current live flow is different.

### Overview

Supports two payment types (Fiat and Stablecoin) for both subscription and redemption. Subscription adds a Notification of Intent (NI) step before order submission.

**Key design principles:**
- **Burn at Accept (Redemption):** Tokens burned at accept stage, NOT at resolve
- **Lock at Submit (Stablecoin Sub):** Stablecoin locked in Delta Manager at submit, refunds automatically on reject
- **NI Step (Subscription only):** Non-binding indication of intent. No SC involved. Auto-accepted by system.
- **No Auto-Fail:** System never auto-rejects expired orders. FA/FM must manually reject.
- **Fiat Refund is Manual:** Fiat rejection refunds handled off-chain via ACME. Stablecoin refunds are automatic on-chain.
- **Direct SC Bypass:** App must listen for events from investors calling Delta Manager directly (bypassing LIP portal).

### Status Flows

```
SUBSCRIPTION:
NI_PENDING → NI_ACCEPTED → SUBMITTED → ORDER_ACCEPTED → RESOLUTION_COMPLETED
                                      ↘ ORDER_REJECTED

REDEMPTION:
SUBMITTED → ORDER_ACCEPTED → RESOLUTION_COMPLETED
          ↘ ORDER_REJECTED
```

---

### SUBSCRIPTION FLOW

#### Step 1 — Notification of Intent (NI)
> Same for Fiat and Stablecoin. No SC call involved.

1. Investor: Connect Wallet → Select Chain → Select Fund → Choose Payment Type → Input Amount → Choose Dealing Date → Submit NI
2. Backend creates NI record (no SC call). Status → **NI_PENDING**
3. System auto-accepts NI. Status → **NI_ACCEPTED**
4. Email to Investor: NI Accepted (cutoff time, action: submit order before deadline)
5. Email to FA/FM: New NI Received (informational only)

Note: FA/FM can still reject at NI_ACCEPTED status.

#### Step 2A — Submit Order (Fiat)
> No money moves on-chain. SC records intent only.

1. Investor reviews Fiat Order Form (amount pre-filled & locked from NI, bank transfer details shown)
2. Investor submits order
3. SC: `requestSubFiat(collateralAmount)` — no money moves, subscription metadata created, `subscriptionId` generated
4. Status → **SUBMITTED**
5. Investor performs bank transfer off-chain (amount must match order)
6. Email to Investor: Order Submitted (bank details, reference code, deadline)
7. Email to FA/FM: Order Submitted (action required: monitor fund bank account)
8. Task created for FA/FM in admin portal

#### Step 2B — Submit Order (Stablecoin)
> Stablecoin immediately deducted and locked in Delta Manager contract.

1. Investor: ERC-20 Approve (Delta Manager to transfer on behalf)
2. Investor reviews Stablecoin Order Form → Submit
3. SC: `requestSubStablecoin(collateralAmountIn, collateralToken)` — stablecoin immediately deducted from investor wallet, locked in Delta Manager, fee routed to `feeRecipient`
4. Status → **SUBMITTED**
5. Email to Investor & FA/FM: Order Submitted (tx hash, stablecoin locked, awaiting acceptance)
6. Task created for FA/FM

#### Step 3A — FA/FM Accept or Reject (Fiat)
> Pre-condition: Fiat must have arrived in fund bank account (FOBA) before accepting.

**Accept (full or partial):**
1. FA/FM reviews order → inputs accepted amount
2. SC: `acceptSubFiat(subID, acceptAmt)` → `acceptSubscription` event emitted
3. Status → **ORDER_ACCEPTED**
4. Unaccepted portion refunded off-chain via ACME (manual)

**Reject:**
1. SC: `rejectSubscription(subID)`
2. Status → **ORDER_REJECTED**
3. Fiat refund handled manually by FA/FM via ACME (no automated refund)

#### Step 3B — FA/FM Accept or Reject (Stablecoin)
> Pre-condition: Stablecoin confirmed locked in Delta Manager. StraitsX needs 2–3 days to convert and remit fiat before dealing date.

**Accept (full or partial):**
1. FA/FM inputs accepted amount + selects On/Off-ramp Partner
2. SC: `acceptSubStablecoin(subID, acceptAmt, assetRecipient)` — accepted stablecoin routed to `assetRecipient` (e.g. StraitsX); unaccepted portion auto-returned to investor wallet on-chain
3. Status → **ORDER_ACCEPTED**; Fireblocks approval required

**Reject:**
1. SC: `rejectSubscription(subID)` — stablecoin auto-returned to investor wallet on-chain
2. Status → **ORDER_REJECTED**

#### Step 4 — Resolve Order (Fiat & Stablecoin)
1. FA/FM posts NAV: `setNavForMultipleEpochs(navScaled, dealingDate)` (can backdate or future-date)
2. FA/FM resolves: `resolveSub(subscriptionId, dealingDate, mintQty)` → `checkKYC` → `deltaToken.mint(investorWallet, mintQty)`
3. Fireblocks approval required
4. Status → **RESOLUTION_COMPLETED**
5. Email to Investor: Subscription Complete — Tokens Minted

**Admin Resolve Form — Fiat:**
Requested Amount (USD) → Received Amount (USD, confirmed in FOBA via ACME) → Fund Fee (USD) → Rejected Amount (USD, if partial) → Fulfilled Amount = Received − Fee − Rejected → Dealing Date → NAV → Estimated Tokens to Mint = Fulfilled ÷ NAV (editable)

**Admin Resolve Form — Stablecoin:**
Requested Amount (USDC) → Indicative Equivalent (USD) → Amount Sent to On/Off-ramp (USDC) → On/Off-ramp Fee → Received Amount (USD, net from partner) → Fund Fee (USD) → Fulfilled Amount = Received − Fee → Dealing Date → NAV → Estimated Tokens to Mint = Fulfilled ÷ NAV (editable)

---

### REDEMPTION FLOW

#### Step 1A — Investor Redeems for Fiat
1. Investor: Connect Wallet → Select Chain → Select Fund → Input Token Amount → Review Fiat Redemption Form → Submit
2. SC: `requestRedeemFiat(tokenAmount)` — no money moves, SC records intent only
3. Status → **SUBMITTED**
4. Email to Investor: Fiat Redemption Submitted; Email to FA/FM: action required

#### Step 1B — Investor Redeems for Stablecoin
1. Investor: Connect Wallet → Input Token Amount + Receiver Wallet Address → ERC-20 Approve → Review → Submit
2. SC: `requestRedeemStablecoin(tokenAmount, receiverWallet)` — tokens NOT burned yet at submit
3. Status → **SUBMITTED**
4. Email to Investor: Stablecoin Redemption Submitted; Email to FA/FM: action required

#### Step 2 — FA/FM Accept or Reject (Same for Fiat & Stablecoin)

**Accept (full or partial):**
1. FA/FM posts NAV for dealing date → inputs token amount to accept → calculates net payable
2. SC: `acceptRedemption(tokenAmount, navPerToken)` — **TOKENS BURNED IMMEDIATELY** — fund fee deducted — net payable USD calculated
3. Fireblocks approval required; Status → **ORDER_ACCEPTED**
4. Email to Investor: Tokens Burned, NAV used, Net Payable USD, payout at resolve step

**Reject:**
1. SC: `rejectRedemption` — tokens returned to investor wallet
2. Status → **ORDER_REJECTED**; Email to Investor: tokens returned automatically

Note: Manual rejection required — admin must reject even if past due date. No auto-fail.

#### Step 3A — Resolve (Fiat)
> No Fireblocks needed. No on-chain token transfer.

1. FA/FM confirms fiat bank transfer sent off-chain
2. SC: `resolveFiatRedemption(redemptionId)` — marks order resolved on-chain, no on-chain money movement
3. Status → **RESOLUTION_COMPLETED**
4. Email to Investor: Fiat Payment Initiated (net payable, bank details, expected arrival, payment reference)

#### Step 3B — Resolve (Stablecoin)
> Fireblocks required. Phase 1: full payout only.

1. FA/FM initiates USDC payout via Fireblocks to investor receiver wallet
2. SC: `resolveStablecoinRedemption(redemptionId, amount, receiverWallet)` — USDC transferred on-chain
3. Fireblocks approval required; Status → **RESOLUTION_COMPLETED**
4. Email to Investor: USDC Payment Sent (amount, receiver wallet, tx hash)

---

### Smart Contract Function Reference

| Function | When Called | Key Effect |
|----------|-------------|------------|
| `requestSubFiat(collateralAmt)` | Sub submit (fiat) | Records intent, no money moves |
| `requestSubStablecoin(collateralAmtIn, collateralToken)` | Sub submit (stablecoin) | Locks stablecoin in contract |
| `acceptSubFiat(subID, acceptAmt)` | FA accepts fiat sub | Records accepted amount |
| `acceptSubStablecoin(subID, acceptAmt, assetRecipient)` | FA accepts stablecoin sub | Routes accepted to on/off-ramp, refunds remainder on-chain |
| `rejectSubscription(subID)` | FA rejects sub | Returns stablecoin (if applicable) |
| `resolveSub(subID, dealingDate, mintQty)` | FA resolves sub | checkKYC → mint tokens |
| `setNavForMultipleEpochs(navScaled, dealingDate)` | FA posts NAV | Sets NAV, supports backdating and future-dating |
| `requestRedeemFiat(tokenAmt)` | Redeem submit (fiat) | Records intent, no token burn |
| `requestRedeemStablecoin(tokenAmt, receiverWallet)` | Redeem submit (stablecoin) | Records intent, no burn yet |
| `acceptRedemption(tokenAmt, navPerToken)` | FA accepts redeem | **Burns tokens immediately**, calculates net payable |
| `rejectRedemption` | FA rejects redeem | Returns tokens to investor |
| `resolveFiatRedemption(redemptionId)` | FA resolves fiat redeem | Marks resolved, no on-chain money |
| `resolveStablecoinRedemption(redemptionId, amt, wallet)` | FA resolves stablecoin redeem | Transfers USDC to investor |
| `deltaToken.mint(wallet, qty)` | Called by resolveSub | Mints tokens to investor wallet |
| `payoutDividendsDeltaToken(recipients[], amounts[])` | Dividend payout | Payout in DELTA tokens |
| `payoutDividendsStablecoin(recipients[], amounts[], collateralToken)` | Dividend payout | Payout in stablecoin |

---

## 6. Supporting Modules

### Token Control Module (Admin)

**Purpose:** Admin-facing interface to manage the full token lifecycle.

**Operations:**
| Operation | Description |
|-----------|-------------|
| **Mint** | Issue tokens to a wallet (token selection, gas fee, reason logging) |
| **Burn** | Destroy tokens from a wallet (irreversibility warning); only for tokens previously clawed back |
| **Clawback** | Reclaim tokens + auto-freeze wallet + compliance reason codes + Fireblocks approval |
| **Transfer** | Move tokens between wallets; routes through Ultra Manager contract |
| **Release Collateral** | Release stuck stablecoin from Delta Manager contract |

**Token Operation Status Flow:**
Requested → Submitted → (Pending FA / Pending Fireblocks / Pending On-chain) → Rejected / Error / Settled

For Error state: resubmit button available → brings back to Submitted state.

**Token Settings (per token):**
- Minimum/maximum transfer amount; transfer limit (period cap); auto-approval threshold
- Token suspension — halts all operations/transfers (confirmation modal + audit trail)
- Audit logs: Operations log + Settings log (both exportable as CSV)

**Transfer Logic (SC Redesign v2):**
- Transfer limits changed from 24-hour per-investor to per-transaction
- ALL transfers must route through Ultra Manager contract (no threshold bypass)
- When FA/FM rejects transfer on Fireblocks → tokens route back to sender
- KYC check required on all token transfers

### NAV Management
- NAV set per epoch via `setNavForMultipleEpochs(navScaled, dealingEpoch)`
- Can set for past, current, or future epochs (subject to `maxFutureEpochsForNav`)
- Retrieve NAV via `getNav(dealingDate)` — converts date internally to epoch

---

## 7. External Integrations

| Partner | Role | Notes |
|---------|------|-------|
| **Fireblocks** | MPC platform for on-chain transaction signing | Required for all on-chain operations (mint, burn, transfer, resolve stablecoin) |
| **StraitsX** | On/Off-ramp partner | Fiat ↔ stablecoin conversion; needs 2–3 days lead time |
| **Wellington** | Treasury manager | Manages fund treasury, initiates bank transfers |
| **ACME** | Banking integration | Integrates with SCB/UOB for payment instructions and transaction confirmation; fiat leg of sub/redemption |
| **Chekk** | KYC/KYB provider | Individual KYC and corporate KYB verification |
| **Chainalysis** | AML screening | Wallet screening with daily re-scan |
| **Azure AD B2C** | Admin portal authentication | OTP-based login |
| **SendGrid** | Email delivery | 45 system templates + 13 dynamic templates |

**Vendor naming rule:** Always reference by role + current name: "the on/off-ramp partner (currently StraitsX)", "the treasury manager (currently Wellington)". Partners may change.

---

## 8. Admin Portal UI Structure

### Sidebar Navigation
```
Requests
  └─ Vault App
       ├─ Subscription
       └─ Redemption
Client
  ├─ Investors (Vault dApp legacy)
  └─ Clients (LIP new)
Admin
  ├─ Manage Admins
  └─ Invite Admin
NAV History
Wallet Management
Task Management
```

### List Table Columns

**Subscription:** Last Updated | Client Name | Asset Name | Funding Method | Subscription Amount | Fulfilled Amount | Fulfilled Tokens | Dealing Date | Order Status

**Redemption:** Last Updated | Client Name | Asset Name | Funding Method | Redemption Token | Requested Amount (USD) | Fulfilled Amount | Net Payable | Dealing Date | Order Status

### Filter Options

**Subscription:** Status (Intent Received / Intent Rejected / Order Submitted / Order Accepted / Order Rejected / Order Resolved) | Funding Method (Stablecoin / Fiat) | Asset | Date range

**Redemption:** Status (Request Submitted / Request Accepted / Request Rejected / Request Resolved) | Funding Method | Asset | Date range

### Order Details Sections (both sub and redemption)
- **Order Information** — amounts, dates, NAV, fees
- **Payment Information** — ACME ID, bank account, status
- **Blockchain Transaction History** — function name, TxHash, date, Fireblocks status
- **Client Information** — classification, name, user ID, status, wallet; "View Details" link
- **Timeline** — chronological events with actor + timestamp + role badge (FA/WD)

### Status Chips

**Subscription (admin):**
| Status | Color |
|--------|-------|
| Intent Received | Green #15803d |
| Intent Rejected | Red #b91c1c |
| Order Submitted | Blue #0369a1 |
| Order Accepted | Green #15803d |
| Order Rejected | Gray #656566 |
| Order Resolved | Gray #656566 |

**Redemption (admin):**
| Status | Color |
|--------|-------|
| Request Submitted | Blue #0369a1 |
| Request Accepted | Green #15803d |
| Request Resolved | Purple #6941c6 |
| Request Rejected | Red #b91c1c |

---

## 9. Investor App (Delta Vault App) UI Reference

### Navigation
- Tabs: Explore | Invest | Transactions | Privacy
- Sub-tabs on Invest: Subscribe | Redeem
- Fund selector dropdown (ULTRA, MG999 variants) + Chain selector + connected wallet address

### Fund Info Card (always visible)
| Field | Example |
|-------|---------|
| Fund ticker | ULTRA |
| Full name | Delta Wellington Ultra Short Treasury On-Chain Fund |
| Fund Size | $120M |
| Latest NAV | $1.50 |
| Dealing Date | Monday – Friday |
| Currency | USD |
| Min. Subscription | $50,000 |
| Max. Subscription | Unlimited |
| Min. Redemption | $50,000 |
| Max. Redemption | $100,000 |

### NI Form Fields (Subscription)
1. Intended Amount — numeric, suffix "USD"; tooltip: "non-binding indication of intent, no funds transferred"
2. Indicative Dealing Date — toggle Standard / Adhoc + computed date; Cutoff: 12:00:00 UTC
3. Funding Method — read-only display
4. Disclaimer banner
5. Submit button

### Redemption Form Fields

**Stablecoin:** Amount (token qty) + Max button + balance display | Funding Method | Receive To: pre-filled wallet + Whitelisted chip | On/Off Ramp Fee: 0.10% | Est. Redeem Amount = token qty × NAV × (1 − fee)

**Fiat:** Amount (token qty) + Max button | Receive To: bank account selector (saved accounts + "Add new bank account") | No on-chain fee displayed

### Transaction History Table
Date | Type | Asset Name | Amount | Funding Method | Status | Order ID | Action (Proceed / View)

### Status Chips (Investor)

**Subscription:**
| Status | Color |
|--------|-------|
| Intent rejected | Red #b91c1c |
| Order submitted | Blue #0369a1 |
| Order accepted | Green #15803d |
| Order rejected | Red #b91c1c |
| Order resolved | Green #15803d |
| Pending dealing | Orange #a16207 |
| Processing | Blue #0369a1 |
| Completed | Green #15803d |

**Redemption:**
| Status | Color |
|--------|-------|
| Request submitted | Neutral |
| Request accepted | Green #15803d |
| Request rejected | Red #b91c1c |
| Request resolved | Green #15803d |
| Processing | Blue #0369a1 |

### Sample Data (used across wireframes)
| Field | Value |
|-------|-------|
| Fund tickers | ULTRA, MG999 |
| NAV | $1.40–$1.50 USD |
| Investor names | Walt Disney (WD), Ryan Ryu (RR), Zisse Nguyen (ZN) |
| Admin | Zisse Nguyen / FA / zisse.nguyen@gem-corp.tech |
| Wallet | `0xe597...662b` |
| Sub Order ID | #SUB_123456 |
| Red Order ID | #RED_123456 |
| ACME ID | txn_0P8M4NWATQP8C |
| On/Off Ramp vendor | StraitsX |
| On/Off Ramp fee | 0.10% |
| Bank | Standard Chartered Bank / ****7890 |
| USDC reserve | 300,000,000 USDC (Ultra Manager contract) |
| Chain | Ethereum (Sepolia on UAT) |
| Token timelines | T+1 BD (burn/mint) / T+3 BD (cash settlement) |
| Dealing cutoff | 12:00:00 UTC |
| Date format | DD/MM/YYYY HH:MM:SS |

---

*Source: Confluence LIBERTY space (PRDs, SC Redesign, Bank Account, RBAC, Token Control, Onboarding), FigJam "Proper Sub. and Redemption Flow", Figma "LIP" wireframes*
*Last updated: 2026-05-19*
