# Budgetary Control & Encumbrance Service Specification

## Overview

The Budgetary Control and Encumbrance Accounting service ensures that organizational spending stays within approved budget limits by reserving funds at various stages of the procurement and financial lifecycle. This aligns with Oracle Fusion Cloud Budgetary Control.

## Data Model (SQLite)

- `control_budgets`: id (UUID), name, status (ACTIVE/ARCHIVED), type (ADVISORY/ABSOLUTE/TRACK_ONLY), currency, start_date, end_date.
- `budget_segments`: id (UUID), budget_id, chart_of_accounts_id, segment_values (JSON).
- `budget_balances`: id (UUID), segment_id, period_id, total_budget, total_encumbrance, total_actual, funds_available.
- `funds_reservations`: id (UUID), source_doc_id, source_type (REQUISITION/PO/INVOICE/JOURNAL), amount, status (RESERVED/LIQUIDATED/FAILED).
- `encumbrance_ledger`: id (UUID), reservation_id, period_id, gl_account_id, amount, debit_credit_flag.
- `budget_overrides`: id (UUID), reservation_id, approved_by, override_reason, date.

## API Endpoints (Axum/gRPC)

- `POST /budgetary-control/check-funds`: Perform a real-time check to see if funds are available for a transaction.
- `POST /budgetary-control/reserve-funds`: Increment encumbrance and decrease funds available for a source document.
- `POST /budgetary-control/liquidate-funds`: Move funds from "Encumbrance" to "Actual" (e.g., when a PO becomes an Invoice).
- `GET /budgetary-control/balances`: Retrieve the current spending status for a specific budget account.
- `POST /budgetary-control/override`: Record an executive approval to exceed a budget limit.
- `GET /budgetary-control/analytics/spending-trends`: View historical and projected spend against budget.

## Inter-service Interaction

- Receives `Requisition_Submitted` from `Procurement` to perform initial funds check.
- Receives `PO_Approved` from `Procurement` to commit a funds reservation.
- Receives `Invoice_Matched` from `Financials` (AP) to liquidate encumbrance and record actual spend.
- Queries `EPM Service` (Planning) for approved budget amounts and versions.
- Emits `Funds_Failed_Alert` to requesters and budget managers.

## Key Logic

- **Funds Reservation Point:** Allowing the organization to decide if funds are "locked" at Requisition time or only when a formal PO is approved.
- **Advisory vs. Absolute Control:** Choosing whether the system should only warn users ("Advisory") or block transactions ("Absolute") when a budget is exceeded.
- **Encumbrance Accounting:** Automatically generating double-entry accounting for commitments (Debit: Encumbrance, Credit: Reserve for Encumbrance).
- **Rollover Logic:** Managing the transition of unused budget and open encumbrances from one fiscal year to the next.
- **Segment-Level Control:** Enforcing budgets at different granularities (e.g., by Department, Project, or specific GL Account).
- **Parallel Approval Integration:** Ensuring that budgetary checks happen simultaneously with business approvals to prevent bottlenecks.
