# Expense Management Service Specification

## Overview

The Expense Management service handles employee expense reporting, mobile receipt scanning, approval workflows, and reimbursement. This aligns with Oracle Fusion Cloud Expenses.

## Data Model (SQLite)

- `expense_reports`: id (UUID), employee_id (link to HCM), description, status (DRAFT/SUBMITTED/APPROVED/REJECTED/PAID), total_amount, currency.
- `expense_items`: id (UUID), report_id, date, expense_type (TRAVEL/MEAL/HOTEL/COMMUTE), amount, tax_amount, receipt_image_url.
- `expense_types`: id (UUID), name, account_id (link to GL), maximum_limit, requires_receipt (BOOL).
- `approvals`: id (UUID), report_id, approver_id (link to Identity), status, comments, timestamp.
- `policy_violations`: id (UUID), item_id, description, severity (WARNING/CRITICAL).
- `credit_card_transactions`: id (UUID), employee_id, transaction_id, merchant, amount, date, status (MATCHED/UNMATCHED).

## API Endpoints (Axum/gRPC)

- `POST /expenses/reports`: Create and submit expense reports.
- `POST /expenses/items`: Add expense items to a report (supports OCR for receipts).
- `GET /expenses/pending-approvals`: List reports awaiting manager approval.
- `POST /expenses/approve-reject`: Update status of a submitted report.
- `GET /expenses/history`: View past employee expense reports and payment status.
- `POST /expenses/cc-sync`: Import and match corporate credit card transactions.

## Inter-service Interaction

- Queries `HCM Service` for employee organizational hierarchy (who approves whose reports).
- Emits `Expense_Approved` event to `Financials` to create an Accounts Payable invoice (for reimbursement).
- Queries `Financials` for chart-of-account segments to correctly categorize expenses.
- Emits `Policy_Breached` events to `Risk Management Service` for audit tracking.

## Key Logic

- **Policy Engine:** Automatically flag items that exceed corporate spending limits based on expense type.
- **OCR Integration:** Use AI to extract merchant name, date, and amount from receipt images.
- **Workflow Routing:** Dynamically route approvals based on report total and employee department.
- **Reimbursement Processing:** Integration with AP to ensure employees are paid back in the next payment cycle.
