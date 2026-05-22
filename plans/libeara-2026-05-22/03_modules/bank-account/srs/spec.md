---
type: srs
feature: bank-account
status: draft
created: 2026-05-22
updated: 2026-05-22
owner: "@hienduong"
version: 0.1.0
changelog:
  - 2026-05-22 | /ba-start | initial draft
---

# SRS — Bank Account Module

**Project:** Libeara Investment Platform (LIP)
**Version:** v0.1.0
**Owner:** @hienduong
**Date:** 2026-05-22

## 1. Purpose and Scope

Đặc tả yêu cầu cho Bank Account Module trên LIP. Module cho phép investors đăng ký và quản lý tài khoản ngân hàng dùng cho fiat transactions (subscription và redemption). Admin có thể approve/reject và thêm tài khoản thay mặt investor.

Out of scope: stablecoin wallet management, Crane platform.

## 2. Functional Requirements

| FR ID | Requirement | Business Value | Source | AC Summary |
| --- | --- | --- | --- | --- |
| FR-bank-001 | Investor có thể thêm bank account với các trường: currency, beneficiary name, bank name, account number, SWIFT, correspondent bank, documents | Cho phép fiat redemption | Knowledge Base §2 | Account submitted → status PENDING APPROVAL |
| FR-bank-002 | Investor có thể xem danh sách tất cả bank accounts đã link kèm status | Transparency | Knowledge Base §2 | Hiển thị đúng status hiện tại |
| FR-bank-003 | Investor có thể update account (triggers re-verification) | Data accuracy | Knowledge Base §2 | Status reset về PENDING APPROVAL sau update |
| FR-bank-004 | Investor có thể xóa account nếu không linked với active investment hoặc pending transaction | Data hygiene | Knowledge Base §2 | Soft-delete — data kept for audit; status → REMOVED |
| FR-bank-005 | Admin có thể approve/reject bank account | Compliance gate | Knowledge Base §2 | Approve → ACTIVE; Reject → INACTIVE |
| FR-bank-006 | Admin có thể request thêm thông tin/documents từ investor | Incomplete submission handling | Knowledge Base §2 | Status → ACTION REQUIRED; investor nhận notification |
| FR-bank-007 | Admin có thể thêm bank account thay mặt investor (Maker-Checker) | Operational support | Knowledge Base §2 | Tạo bởi admin → PENDING APPROVAL → ACTIVE/INACTIVE |
| FR-bank-008 | Approve/reject triggers task trong Task Management | Workflow integration | Knowledge Base §2 | Task auto-created khi status thay đổi cần review |

## 3. Non-Functional Requirements

| NFR ID | Category | Requirement | Trigger / Gate |
| --- | --- | --- | --- |
| NFR-bank-001 | Security | Tài liệu upload (documents) phải được lưu trữ an toàn, không public-accessible | Trước khi build upload feature |
| NFR-bank-002 | Audit | Full audit log cho mọi status transition và admin action | Bắt buộc trước go-live |
| NFR-bank-003 | Compliance | Soft-delete only — data không được xóa vĩnh viễn | Bắt buộc, MAS compliance |
| NFR-bank-004 | Performance | Danh sách bank accounts load < 2s với ≤ 50 accounts | Trước UAT |

## 4. Business Rules

| BR ID | Rule |
| --- | --- |
| BR-bank-001 | Account chỉ có thể xóa nếu không linked với active investment hoặc pending transaction |
| BR-bank-002 | Edit active account (ACTIVE status) → tự động reset về PENDING APPROVAL |
| BR-bank-003 | Non-active account (INACTIVE, ACTION REQUIRED) → xóa tự động không cần admin approval |
| BR-bank-004 | Active account removal → phải qua PENDING REMOVE → admin approve → REMOVED |
| BR-bank-005 | Investor không phản hồi ACTION REQUIRED → admin có thể chuyển sang INACTIVE |

## 5. Status Lifecycle

```
Client submits          → PENDING APPROVAL
Admin approves          → ACTIVE
Admin needs more info   → ACTION REQUIRED
Client resubmits        → PENDING APPROVAL
Client no response      → INACTIVE
Client edits ACTIVE     → PENDING APPROVAL
Client requests remove (ACTIVE) → PENDING REMOVE → REMOVED (admin approval)
Non-active remove       → REMOVED (automatic)
Admin adds on behalf    → PENDING APPROVAL → ACTIVE / INACTIVE
```

## 6. Portal Matrix

| Portal ID | Portal | Target Actor | Owned Screens |
| --- | --- | --- | --- |
| PORTAL-INVESTOR | LIP Investor App | Investor (Individual / Corporate) | Bank Account List, Add/Edit Bank Account |
| PORTAL-ADMIN | LIP Admin Portal | Fund Admin (FA), Super Admin | Bank Account Management, Approve/Reject |

## 7. Screen Inventory

| Screen ID | Portal | Screen | Complexity | Wireframe |
| --- | --- | --- | --- | --- |
| SCR-bank-01 | PORTAL-INVESTOR | Bank Account List | Low | No |
| SCR-bank-02 | PORTAL-INVESTOR | Add Bank Account Form | Medium | Yes |
| SCR-bank-03 | PORTAL-INVESTOR | Edit Bank Account Form | Medium | No (reuse Add) |
| SCR-bank-04 | PORTAL-ADMIN | Bank Account Management List | Medium | No |
| SCR-bank-05 | PORTAL-ADMIN | Bank Account Detail + Approve/Reject | Medium | Yes |
| SCR-bank-06 | PORTAL-ADMIN | Add Bank Account on Behalf (Maker) | Medium | No |

## 8. Open Questions

1. [ ] Documents upload: file types và size limits được chấp nhận là gì?
2. [ ] Correspondent bank field — bắt buộc hay optional?
3. [ ] Notification channel khi status thay đổi: email only hay cả in-app?
4. [ ] Maker-Checker: ai là Checker khi admin thêm account thay investor?
