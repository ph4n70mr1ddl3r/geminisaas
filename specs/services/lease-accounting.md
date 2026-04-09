# Lease Accounting Service Specification

## Overview

The Lease Accounting service manages the lifecycle of lease contracts for assets (e.g., real estate, vehicles, equipment) in compliance with IFRS 16 and ASC 842 standards. This includes calculating right-of-use (ROU) assets and lease liabilities. This aligns with Oracle Fusion Cloud Lease Accounting.

## Data Model (SQLite)

- `lease_contracts`: id (UUID), name, status (DRAFT/ACTIVE/EXPIRED/TERMINATED), lessee_id, lessor_id, start_date, end_date, interest_rate.
- `lease_payments`: id (UUID), contract_id, due_date, amount, type (FIXED/VARIABLE/SERVICE), frequency.
- `lease_assets`: id (UUID), contract_id, asset_id (optional link to Fixed Assets), rou_asset_value, status.
- `lease_liabilities`: id (UUID), contract_id, principal_balance, interest_balance, period_id.
- `lease_amortization_schedules`: id (UUID), contract_id, period_id, depreciation_amount, interest_amount, liability_reduction.

## API Endpoints (Axum/gRPC)

- `POST /leases`: Create a new lease contract and calculate the ROU asset.
- `POST /leases/{id}/activate`: Transition a lease to active status and generate the full amortization schedule.
- `POST /leases/{id}/amend`: Modify lease terms (e.g., extension, price change) and recalculate balances.
- `GET /leases/reports/disclosure`: Generate audit-ready reports for lease disclosure requirements.
- `POST /leases/generate-invoices`: Create payment requests (AP) for due lease installments.

## Inter-service Interaction

- Emits `Lease_Capitalization_Posted` to `Financials` (and optionally `Fixed Assets`) to record the ROU asset and initial liability.
- Emits `Monthly_Amortization_Posted` to `Financials` to record interest and depreciation.
- Emits `Lease_Payment_Due` to `Procurement` (AP) to trigger the vendor payment process.
- Queries `Financials` for market interest rates or incremental borrowing rates.

## Key Logic

- **Present Value Calculation:** Automatically calculating the present value of future lease payments to determine initial liability.
- **Amortization Engine:** Generating period-by-period schedules for interest expense and liability reduction.
- **Contract Modifications:** Handling complex reassessments when lease terms change, ensuring the ROU asset is correctly adjusted.
- **Standard Switching:** Capability to report under both IFRS 16 (Finance Lease model) and ASC 842 (Operating and Finance Lease models).
