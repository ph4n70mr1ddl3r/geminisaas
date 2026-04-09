# Fund Accounting Service Specification

## Overview

The Fund Accounting service provides specialized financial management for non-profit organizations and government agencies, enabling the tracking of "restricted" and "unrestricted" funds. It ensures that expenditures are aligned with specific donor or legislative mandates. This aligns with Oracle Fusion Cloud Fund Accounting.

## Data Model (SQLite)

- `fund_definitions`: id (UUID), fund_code, name, type (RESTRICTED/UNRESTRICTED/ENDOWMENT), status (ACTIVE/CLOSED).
- `fund_restrictions`: id (UUID), fund_id, restriction_text, expiry_date, donor_id (link to CDM).
- `fund_balances`: id (UUID), fund_id, gl_account_id, period_id, opening_balance, total_debit, total_credit, ending_balance.
- `inter_fund_transfers`: id (UUID), from_fund_id, to_fund_id, amount, reason_code, approval_status.
- `encumbrance_fund_ledger`: id (UUID), fund_id, reservation_id (link to Budgetary Control), amount, status.
- `fund_spending_guardrails`: id (UUID), fund_id, category_id, max_spending_pct, status.

## API Endpoints (Axum/gRPC)

- `POST /fund-accounting/funds`: Create a new restricted or unrestricted fund.
- `GET /fund-accounting/balances/{fund_id}`: Retrieve real-time financial balances for a specific fund.
- `POST /fund-accounting/transfer`: Move money between funds while maintaining balancing entries.
- `GET /fund-accounting/reports/donor-disclosure`: Generate financial statements for a specific donor.
- `POST /fund-accounting/validate-spend`: Ensure a proposed expense is compliant with fund restrictions.
- `GET /fund-accounting/analytics/burn-rate`: Analyze how quickly restricted funds are being consumed.

## Inter-service Interaction

- Receives `Journal_Posted` from `Financials (GL)` to update fund-specific balances.
- Queries `Budgetary Control` for funds-available checks against specific funding sources.
- Emits `Restriction_Violation_Alert` to the organization's controller or auditor.
- Queries `Grants Management` for federal or private grant funding rules.
- Emits `Fund_Exhausted_Signal` to `Procurement` to block further spending.

## Key Logic

- **Self-Balancing Funds:** Ensuring that every fund's debits and credits always equal, even during inter-fund transactions.
- **Pooling & Distribution:** Allowing multiple funds to pool cash for investment while accurately distributing earnings back to individual funds.
- **Multi-Year Funds:** Supporting funds that span multiple fiscal years without the need for annual closing processes.
- **Indirect Cost Recovery:** Automatically calculating and charging "overhead" from a restricted fund to the general fund.
- **Donor Compliance Monitoring:** Forcing specific justifications for expenses charged to a highly restricted endowment fund.
- **Fund-Level Reporting:** Generating distinct Balance Sheets and Income Statements for each fund or group of funds.
