# Trade Management Service Specification

## Overview

The Trade Management service handles the settlement of customer claims and deductions, specifically for industries like consumer goods (CPG) where complex promotional agreements are common. This aligns with Oracle Fusion Cloud Trade Management and Oracle Channel Revenue Management.

## Data Model (SQLite)

- `trade_promotions`: id (UUID), name, status (DRAFT/ACTIVE/EXPIRED), type (OFF_INVOICE/SCAN_DOWN/BILL_BACK), customer_id, product_id, discount_amount, start_date, end_date.
- `customer_claims`: id (UUID), customer_id, source_id (link to AR/Logistics), type (DEDUCTION/OVERPAYMENT/CLAIM), amount, reason_code, status (OPEN/RESEARCH/APPROVED/REJECTED), claim_date.
- `claim_line_items`: id (UUID), claim_id, invoice_id, amount_claimed, promotion_id (optional), description.
- `trade_settlements`: id (UUID), claim_id, type (CREDIT_MEMO/CHECK/OFFSET), amount, status, date.
- `accrual_ledger`: id (UUID), promotion_id, amount_accrued, amount_paid, period_id.
- `deduction_resolutions`: id (UUID), claim_id, resolution_type (WRITE_OFF/CHARGEBACK/CREDIT), amount, status.

## API Endpoints (Axum/gRPC)

- `POST /trade/promotions`: Create a new promotional agreement for a customer.
- `POST /trade/claims`: Record a new customer claim or deduction.
- `GET /trade/claims/search`: Search for open claims based on customer, amount, or status.
- `POST /trade/claims/{id}/settle`: Resolve a claim via credit memo or payment.
- `GET /trade/accruals/{promotion_id}`: View the current accrual balance for a specific promotion.
- `POST /trade/claims/match-promotion`: Automatically link a claim to an active trade promotion based on rules.

## Inter-service Interaction

- Receives `Deduction_Detected` event from `Financials` (AR) when a customer short-pays an invoice.
- Emits `Credit_Memo_Requested` to `Financials` (AR) to settle a valid claim.
- Queries `Order Management / Commerce` for historical sales data to validate "scan-back" promotions.
- Emits `Promotion_Performance_Updated` to `Marketing`.
- Queries `Logistics Service` for proof-of-delivery (POD) if a claim is for shipping damage.

## Key Logic

- **Promotion Accrual:** Automatically setting aside funds (accruing) for future claims based on current sales volume and promotion terms.
- **Automated Claim Matching:** Link a customer's short-payment (deduction) to an active promotion based on product and date.
- **Dispute Research Workflow:** Providing a structured workspace for collectors and sales reps to investigate why a customer didn't pay in full.
- **Bill-Back Processing:** Managing cases where a customer pays in full but later requests a refund for promotional performance.
- **Offsetting Logic:** Automatically applying overpayments to open deductions to clean up the customer's account.
- **Audit Trails for Deductions:** Ensuring that every claim resolution is documented and approved to meet internal controls (ICFR).
