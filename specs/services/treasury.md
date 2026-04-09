# Treasury Service Specification

## Overview

The Treasury service manages cash positioning, forecasting, bank communications, and financial risk (debt, investments, and hedging). This aligns with Oracle Fusion Cloud Cash Management and Treasury.

## Data Model (SQLite)

- `bank_accounts`: id (UUID), name, iban/account_number, currency, current_balance, status.
- `bank_statements`: id (UUID), bank_account_id, period_id, start_balance, end_balance, file_reference.
- `cash_forecasts`: id (UUID), name, type (SHORT_TERM/LONG_TERM), version, period_id, amount.
- `investments`: id (UUID), type (CD/STOCK/BOND), principal, interest_rate, maturity_date, status.
- `debt_instruments`: id (UUID), type (LOAN/BOND_ISSUED), principal, interest_rate, payment_schedule_id.
- `bank_transactions`: id (UUID), bank_account_id, date, amount, description, reconciled (BOOL), gl_journal_id.

## API Endpoints (Axum/gRPC)

- `GET /treasury/cash-position`: Get real-time aggregated cash balance across all bank accounts.
- `POST /treasury/bank-statement/upload`: Ingest a bank statement (e.g., SWIFT MT940, BAI2) for reconciliation.
- `POST /treasury/reconcile`: Match bank transactions with internal GL journal entries.
- `GET /treasury/forecast`: Generate a cash forecast based on AP/AR schedules and planned investments.
- `POST /treasury/investments`: Record a new short-term investment.

## Inter-service Interaction

- Queries `Financials` for AP (Payables) and AR (Receivables) schedules to feed cash forecasting.
- Receives `Payment_Made` and `Payment_Received` events from `Financials` to update internal cash records.
- Emits `Cash_Shortfall_Alert` to management when forecasted cash drops below a threshold.
- Emits `Bank_Fees_Posted` to `Financials` for ledger recording.

## Key Logic

- **Auto-Reconciliation:** Rule-based matching of bank statement lines to GL transactions (e.g., by amount, date, and reference).
- **Liquidity Management:** Sweeping or pooling cash from subsidiary accounts into a master concentration account.
- **Foreign Exchange (FX) Risk:** Tracking exposure to currency fluctuations and suggesting hedging strategies.
- **Debt Amortization:** Automatically calculating interest and principal payments for loans.
