# Receipt Accounting Service Specification

## Overview

The Receipt Accounting service manages the accounting for goods and services as they are received, handling accruals, uninvoiced receipts, and reconciliation with payables. This aligns with Oracle Fusion Cloud Receipt Accounting.

## Data Model (SQLite)

- `receiving_transactions`: id (UUID), source_id (PO/ASN), item_id, quantity_received, unit_price, status (PENDING/ACCOUNTED).
- `receipt_accruals`: id (UUID), transaction_id, accrual_account_id, amount, status (ACTIVE/CLEARED).
- `accrual_clearing_logs`: id (UUID), receipt_id, invoice_id, amount_cleared, variance_amount, timestamp.
- `landed_cost_adjustments_receipt`: id (UUID), receipt_id, charge_type, amount, effective_date.
- `receipt_accounting_distributions`: id (UUID), receipt_id, gl_account_id, amount, debit_credit_flag.

## API Endpoints (Axum/gRPC)

- `POST /receipt-accounting/process`: Generate accrual accounting entries for new physical receipts.
- `POST /receipt-accounting/match-invoice`: Reconcile an AP invoice against a physical receipt to clear the accrual.
- `GET /receipt-accounting/accrual-balance`: Retrieve the "Uninvoiced Receipt" balance for a period.
- `POST /receipt-accounting/clear-anomalies`: Manually clear old accruals that will never be invoiced (e.g., lost paperwork).
- `GET /receipt-accounting/reports/period-end`: Generate reports for month-end accrual reconciliation.

## Inter-service Interaction

- Receives `Receipt_Confirmed` from `Procurement/Inventory` to trigger initial accrual.
- Receives `Invoice_Matched` from `Financials` (AP) to clear the accrual.
- Emits `Accrual_Journal_Entries` to `Financials` (GL).
- Queries `Landed Cost Management` for estimated and actual freight/duty charges.
- Emits `Accrual_Analysis_Signal` to the AI Assurance Advisor.

## Key Logic

- **Accrual at Receipt:** Recording a liability (Accrued Receipts) the moment goods arrive, ensuring financial statements reflect the commitment before the invoice is received.
- **Three-Way Matching Support:** Providing the data backbone for matching PO, Receipt, and Invoice.
- **Accrual Clearing:** Automatically reversing the accrual when the AP invoice is posted, moving the liability from "Accrued" to "Accounts Payable."
- **Retroactive Adjustments:** Handling cases where the receipt quantity or price is corrected after the accounting period has closed.
- **Intercompany Accruals:** Specialized logic for tracking uninvoiced transfers between internal business units.
- **AI-Driven Anomaly Detection:** Flagging accruals that have been open for an unusually long time compared to historical vendor patterns.
