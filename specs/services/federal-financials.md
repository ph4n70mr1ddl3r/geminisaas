# Federal Financials Service Specification

## Overview

The Federal Financials service provides specialized accounting and reporting for U.S. Federal Government agencies, ensuring compliance with Treasury regulations, GTAS, and prompt payment acts. This aligns with Oracle Fusion Cloud Federal Financials.

## Data Model (SQLite)

- `federal_budgets`: id (UUID), fund_code, treasury_symbol, budget_fiscal_year, status, total_authority.
- `federal_apportionments`: id (UUID), budget_id, quarter, amount, status (APPROVED/AVAILABLE).
- `ussgl_accounts`: id (UUID), account_number, name, category (PROPRIETARY/BUDGETARY), status.
- `federal_obligations`: id (UUID), source_doc_id, ussgl_account_id, amount, status (OBLIGATED/PAID).
- `treasury_confirmations`: id (UUID), check_number, payment_date, amount, schedule_id, confirmed_status.
- `gtas_reporting_data`: id (UUID), period_id, trial_balance_json, status (VALIDATED/SUBMITTED).

## API Endpoints (Axum/gRPC)

- `POST /federal/budgets`: Record a new federal budget authority or apportionment.
- `POST /federal/obligations`: Commit a new federal obligation against a treasury symbol.
- `POST /federal/treasury-confirmation`: Record confirmation of a payment from the U.S. Treasury.
- `GET /federal/gtas/validate`: Run validation rules for Governmentwide Treasury Account Symbol Adjusted Trial Balance (GTAS).
- `POST /federal/pay-act/check`: Verify compliance with the Prompt Payment Act for an invoice.
- `GET /federal/reports/sf133`: Generate the Report on Budget Execution and Budgetary Resources.

## Inter-service Interaction

- Queries `Financials Service` for master GL entries and proprietary accounting.
- Emits `Treasury_Payment_Required` event to `Treasury` for schedule generation.
- Receives `Invoice_Matched` from `Financials` (AP) to trigger federal payment logic.
- Queries `Identity Service` for warrants and signing authority.
- Emits `GTAS_Alert` to the agency controller if trial balance rules fail.

## Key Logic

- **Dual-Column Accounting:** Simultaneously recording Proprietary (accrual) and Budgetary (obligation) accounting entries for every transaction.
- **USSGL Compliance:** Enforcing the United States Standard General Ledger (USSGL) account structure and transaction rules.
- **Treasury Symbol Tracking:** Ensuring that every dollar spent is linked to a specific Treasury Account Symbol (TAS) and Budget Fiscal Year (BFY).
- **GTAS Validation:** Automatically running hundreds of Treasury-defined edits on the trial balance before submission.
- **Prompt Payment Act:** Automatically calculating interest penalties if a vendor invoice is not paid within the mandated timeframe (usually 30 days).
- **IPAC Integration:** Handling Intra-governmental Payment and Collection (IPAC) transactions between federal agencies.
