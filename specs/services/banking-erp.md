# Banking ERP Service Specification

## Overview

The Banking ERP service provides specialized financial management for banking and financial institutions, including multi-ledger accounting, liquidity management, and localized regulatory reporting. This aligns with Oracle Fusion Cloud for Financial Services.

## Data Model (SQLite)

- `bank_ledger_accounts`: id (UUID), account_number, type (ASSET/LIABILITY/EQUITY), currency, status, branch_id.
- `daily_bank_balances`: id (UUID), account_id, date, closing_balance, average_balance_ytd.
- `banking_transactions`: id (UUID), source_system (CORE_BANKING/FX/PAYMENTS), transaction_type, amount, currency, status, value_date.
- `multi_currency_settlements`: id (UUID), from_currency, to_currency, exchange_rate, amount, settlement_date, counterparty_id.
- `banking_regulatory_reports`: id (UUID), report_name (BASEL_III/IFRS_9), period_id, status, file_url.
- `liquidity_buffers`: id (UUID), currency, required_reserve, actual_reserve, surplus_deficit.

## API Endpoints (Axum/gRPC)

- `POST /banking-erp/transactions/ingest`: Ingest high-volume transactions from core banking systems.
- `GET /banking-erp/balances/daily`: Retrieve closing and average daily balances for a specific branch or entity.
- `POST /banking-erp/settle/fx`: Execute a multi-currency settlement between internal or external accounts.
- `GET /banking-erp/liquidity/check`: Analyze real-time liquidity against regulatory requirements.
- `POST /banking-erp/reports/generate`: Create localized regulatory filings for central banks.
- `GET /banking-erp/branch-performance`: View profitability and balance trends by branch.

## Inter-service Interaction

- Queries `Financials Service` for master GL structures and corporate-wide consolidations.
- Emits `Liquidity_Threshold_Violated` alert to the Treasury and Risk teams.
- Receives `FX_Rate_Update` from `Treasury` to value multi-currency balances.
- Queries `Identity Service` for branch manager and regional controller permissions.
- Emits `Consolidated_Trial_Balance` to `Financial Consolidation (FCCS)`.

## Key Logic

- **Average Daily Balance (ADB):** Automatically calculating average balances for interest and regulatory reporting purposes.
- **High-Volume Ingestion:** Processing millions of transactions daily from core systems into the sub-ledger and GL.
- **Multi-Book Accounting:** Maintaining multiple sets of books simultaneously (e.g., Local GAAP, IFRS, and Tax).
- **Inter-Branch Settlement:** Automatically generating balancing entries when funds move between branches.
- **Regulatory Stress Testing:** Simulating liquidity and capital adequacy under various economic scenarios (Basel III).
- **Value-Dating:** Handling transactions that are recorded today but have an effective date in the past or future.
