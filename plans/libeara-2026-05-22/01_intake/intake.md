---
type: intake
status: completed
created: 2026-05-22
updated: 2026-05-22
owner: "@hienduong"
source_file: "00_inputs/references/Libeara Knowledge.md"
---

# Phiếu tiếp nhận yêu cầu (Intake Form)

## Thông tin dự án (Project Information)

| Trường (Field) | Giá trị (Value) |
| --- | --- |
| Tên dự án (Project name) | Libeara Investment Platform (LIP) |
| Ngày (Date) | 2026-05-22 |
| Người yêu cầu (Requester) | @hienduong |
| Tài liệu gốc (Source file) | `00_inputs/references/Libeara Knowledge.md` (last updated 2026-05-19) |

## Bối cảnh kinh doanh (Business Context)

### Vấn đề cần giải quyết (Problem Statement)

Libeara là tokenization platform (incubated by SC Ventures / Standard Chartered) cho regulated fund managers. Investors subscribe và redeem tokenized funds bằng fiat hoặc stablecoin. Platform đang trong giai đoạn active development — cần BA documentation đầy đủ cho các module đang build và sắp build.

Vấn đề cốt lõi: hệ thống cũ (Delta Vault dApp) không xử lý đúng mô hình FOP (Free of Payment) — tokens di chuyển on-chain, tiền mặt settle off-chain qua ngân hàng. Redesign toàn bộ sub/redemption flow sang Delta Manager V2 để hỗ trợ FOP đúng cách.

### Mục tiêu kinh doanh (Business Goals)

1. Hoàn thiện BA documentation cho Subscription & Redemption Redesign (Delta Manager V2) — PRD & Design đã có, cần SRS/stories
2. Kickoff và document Token Management Module
3. Kickoff và document ACME Integration (banking fiat leg)
4. Document Email Notifications system
5. Dùng knowledge base này làm context nền cho mọi BA engagement trên platform Libeara/LIP

### Các bên liên quan (Stakeholders Mentioned)

| Tên / Vai trò | Mức quan tâm / Ảnh hưởng | Ghi chú |
| --- | --- | --- |
| Investor (Individual / Corporate) | High / High | End user của LIP platform |
| Fund Admin (FA) | High / High | Operational actions: accept/reject orders, resolve |
| Fund Manager (FM) | High / High | Review/approval per fund |
| Super Admin | High / High | Full platform control, member/role management |
| Ops | Medium / High | KYC/KYB processing, onboarding |
| SC Ventures / Standard Chartered | High / High | Platform owner, MAS-licensed |
| StraitsX | Medium / Medium | On/off-ramp partner (fiat ↔ stablecoin) |
| Wellington | Medium / Medium | Treasury manager |
| ACME (SCB/UOB) | Medium / High | Banking integration, fiat leg |
| Chekk | Low / Medium | KYC/KYB provider |
| Fireblocks | Low / High | MPC platform, required for all on-chain ops |
| Chainalysis | Low / Medium | AML screening |

## Yêu cầu thô (Raw Requirements)

1. **Subscription & Redemption Redesign (Delta Manager V2):** Redesign toàn bộ sub/redemption flow để hỗ trợ FOP. Thêm Notification of Intent (NI) step trước order submission. Stablecoin locking và automatic refund logic. Burn at Accept (Redemption). Lock at Submit (Stablecoin Sub). No Auto-Fail — admin phải manually reject.
2. **LIP Migration Script:** Migrate existing Delta (Vault dApp) users sang LIP — user data, wallet associations, transaction history.
3. **XRPL Integration:** Thêm XRP Ledger làm supported chain.
4. **Solana SC Audit Fixes:** Address critical và medium findings từ Solana smart contract security audit.
5. **Token Management Module:** Admin-facing interface quản lý full token lifecycle (Mint, Burn, Clawback, Transfer, Release Collateral).
6. **ACME Integration:** Banking integration với SCB/UOB cho payment instructions và transaction confirmation. Required cho fiat leg của sub/redemption.
7. **Email Notifications:** Comprehensive email notification system cho mọi investor và admin touchpoints.
8. **2FA Implementation:** Two-factor authentication cho platform.
9. **Ensemble 2.0 (HKMA):** HKMA portal enhancements — 7 requirements, targeting June 2026.
10. **Ultra Bridge (Delta App):** Cross-chain bridge cho Delta tokens. Plan A vs Plan B — product decision pending.
11. **Libby / LibearaGPT:** AI LLM assistant. Under product review.
12. **Crane Solana Expansion:** Deploy CAMC funds (Gold ETFs) on Solana. Target: end June 2026.

## Màn hình và giao diện (Screens and UI)

| Màn hình / Thành phần | Mô tả | Ghi chú |
| --- | --- | --- |
| Admin Portal — Subscription List | Danh sách sub orders với filter/sort | Columns: Last Updated, Client Name, Asset Name, Funding Method, Sub Amount, Fulfilled Amount, Fulfilled Tokens, Dealing Date, Order Status |
| Admin Portal — Redemption List | Danh sách redemption orders | Columns: Last Updated, Client Name, Asset Name, Funding Method, Redemption Token, Requested Amount, Fulfilled Amount, Net Payable, Dealing Date, Order Status |
| Admin Portal — Order Details | Chi tiết order (sub hoặc redemption) | Sections: Order Info, Payment Info, Blockchain Tx History, Client Info, Timeline |
| Admin Portal — Token Management | Mint, Burn, Clawback, Transfer, Release Collateral | Status flow: Requested → Submitted → Pending FA/Fireblocks/On-chain → Rejected/Error/Settled |
| Admin Portal — Member Management | Add/Edit/Delete members, assign roles | Force logout khi disable |
| Admin Portal — Role Management | Create/Edit/Duplicate/Delete roles | Edit role forces logout of all affected users |
| Admin Portal — Bank Account (Admin view) | Approve/reject bank accounts, add on behalf | Maker-Checker flow |
| Admin Portal — NAV History | Set NAV per epoch | Supports backdating và future-dating |
| Admin Portal — Wallet Management | View investor wallets, whitelist status | Chainalysis AML screening |
| Admin Portal — Task Management | View/assign/complete tasks | Linked to sub/redemption orders và bank account requests |
| Investor App (Delta Vault dApp) — Subscribe | NI form + Order form (Fiat/Stablecoin) | NI: non-binding, no SC call. Order: SC call |
| Investor App — Redeem | Redemption form (Fiat/Stablecoin) | Fiat: bank account selector. Stablecoin: receiver wallet |
| Investor App — Transaction History | Danh sách transactions với status chips | Date, Type, Asset, Amount, Funding Method, Status, Order ID, Action |
| Investor App — Bank Account (Client view) | Add/Update/Delete bank accounts | Submit for admin verification |
| Onboarding — Individual | Basic Info → KYC via Chekk → Approval | Status: PENDING → ACTIVE / SUSPENDED |
| Onboarding — Corporate | Basic Corporate + Authorized Person → KYB → Approval | Main Account approved first, then User separately |

## Quy trình và luồng công việc (Processes and Workflows)

| Quy trình / Luồng | Mô tả | Ghi chú |
| --- | --- | --- |
| Subscription Flow (Fiat) | NI → Submit Order (no money moves) → FA Accept/Reject → Resolve (mint tokens) | Pre-condition accept: fiat must arrive in FOBA |
| Subscription Flow (Stablecoin) | NI → Submit Order (stablecoin locked) → FA Accept/Reject (auto-refund on reject) → Resolve (mint tokens) | Fireblocks required at accept và resolve |
| Redemption Flow (Fiat) | Submit → FA Accept (tokens burned immediately) → Resolve (fiat transfer off-chain) | No Fireblocks at resolve |
| Redemption Flow (Stablecoin) | Submit → FA Accept (tokens burned immediately) → Resolve (USDC transferred on-chain) | Fireblocks required at resolve |
| Individual Onboarding | Basic Info → Ops creates KYC in Chekk → KYC review → Approve/Suspend | |
| Corporate Onboarding | Basic Corporate + Auth Person → Ops creates KYB → Chekk verification → Approve | Main Account first, then User |
| Bank Account Lifecycle | Submit → PENDING APPROVAL → ACTIVE / ACTION REQUIRED / INACTIVE | Edit active account triggers re-verification |
| Token Operations | Requested → Submitted → Pending FA/Fireblocks/On-chain → Settled/Rejected/Error | Error state: resubmit available |
| NAV Management | Set NAV per epoch via setNavForMultipleEpochs | Supports past, current, future epochs |

## Ràng buộc và giả định (Constraints and Assumptions)

### Ràng buộc (Constraints)

- MAS-licensed platform — compliance với MAS regulations bắt buộc
- FOP (Free of Payment) là core design principle — tokens on-chain, cash off-chain
- Fireblocks required cho mọi on-chain operations (mint, burn, transfer, resolve stablecoin)
- Crane là separate product — KHÔNG mix với Delta/LIP documentation
- Smart contracts: 3 per fund per chain (KYC Contract, Delta Token ERC-20, Delta Manager V2)
- Supported chains: Ethereum, Solana, Avalanche, Arbitrum (XRPL in progress)
- No Auto-Fail: system không tự reject expired orders — FA/FM phải manually reject
- Fiat refund là manual (via ACME); stablecoin refund là automatic on-chain
- Vendor naming rule: luôn reference by role + current name (e.g., "on/off-ramp partner (currently StraitsX)")
- StraitsX cần 2–3 ngày lead time để convert và remit fiat trước dealing date
- Burn at Accept (Redemption) — tokens burned tại accept stage, KHÔNG phải resolve
- Lock at Submit (Stablecoin Sub) — stablecoin locked tại submit, refund tự động khi reject

### Giả định (Assumptions)

- Knowledge base này (last updated 2026-05-19) là source of truth cho platform state hiện tại
- PRD & Design đã complete cho Sub/Redemption Redesign — BA engagement sẽ dùng làm upstream
- Token Management Module và ACME Integration đang pending kickoff — cần BA documentation từ đầu
- Platform dùng tiếng Anh cho UI và technical artifacts; BA artifacts có thể tiếng Việt

## Tuân thủ và quy định (Compliance and Regulatory Needs)

- **MAS (Monetary Authority of Singapore):** Platform MAS-licensed — mọi feature phải comply
- **AML (Anti-Money Laundering):** Chainalysis daily wallet re-scan cho tất cả wallets
- **KYC/KYB:** Chekk provider — Individual KYC và Corporate KYB bắt buộc trước khi ACTIVE
- **RBAC:** Permissions enforced tại cả UI layer (screen visibility) và API layer
- **Audit Trail:** Full activity log cho member management, bank account, token operations
- **Data Retention:** Bank accounts soft-deleted (REMOVED status), data kept for audit
- **Fireblocks MPC:** Required cho on-chain signing — security compliance

## Câu hỏi mở (Open Questions)

1. Scope cụ thể của BA engagement này là gì? Knowledge base covers toàn bộ platform — cần xác định module nào được prioritize trước (Sub/Redemption V2, Token Management, ACME, hay toàn bộ?)
2. PRD & Design cho Sub/Redemption Redesign có thể share không? Sẽ dùng làm upstream artifact.
3. Output language cho artifacts: tiếng Anh hay tiếng Việt?
4. Engagement mode: lite / hybrid / formal?
5. Ensemble 2.0 (HKMA) — 7 requirements cụ thể là gì? Targeting June 2026 có nghĩa là urgent.
6. Ultra Bridge Plan A vs Plan B — product decision pending, có timeline chốt không?

## Ghi chú phân tích (Parsing Notes)

- Document là onboarding reference, không phải requirements request cụ thể — scope lock cần clarification
- Sub/Redemption Redesign là initiative lớn nhất, có PRD & Design sẵn → ưu tiên cao nhất
- Token Management và ACME Integration đang pending kickoff → cần BA documentation từ đầu
- Crane được đề cập nhưng explicitly out of scope cho LIP/Delta documentation
- Smart contract function reference (Section 5) là technical detail — dùng làm constraint, không phải FR
- Sample data (Section 9) hữu ích cho wireframe/screen spec sau này
