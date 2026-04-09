# Treasury Service Specification

## Overview

The Treasury service manages cash positioning, forecasting, bank communications, and financial risk (debt, investments, and hedging). This aligns with Oracle Fusion Cloud Cash Management and Treasury, including 2026 features like embedded banking and real-time AI-driven cash positioning.

## Data Model (SQLite)

- `bank_accounts`: id (UUID), name, iban/account_number, currency, current_balance, status.
- `bank_statements`: id (UUID), bank_account_id, period_id, start_balance, end_balance, file_reference.
- `cash_forecasts`: id (UUID), name, type (SHORT_TERM/LONG_TERM), version, period_id, amount.
- `investments`: id (UUID), type (CD/STOCK/BOND), principal, interest_rate, maturity_date, status.
- `debt_instruments`: id (UUID), type (LOAN/BOND_ISSUED), principal, interest_rate, payment_schedule_id.
- `bank_transactions`: id (UUID), bank_account_id, date, amount, description, reconciled (BOOL), gl_journal_id.
- `embedded_banking_connections`: id (UUID), bank_name (e.g., BoA), status (ACTIVE/LINKED), credentials_vault_id, last_sync.
- `fx_hedge_contracts`: id (UUID), currency_pair, strike_price, maturity_date, amount, status.

## API Endpoints (Axum/gRPC)

- `GET /treasury/cash-position`: Get real-time aggregated cash balance across all bank accounts using AI-driven position modeling.
- `POST /treasury/bank-statement/upload`: Ingest a bank statement (e.g., SWIFT MT940, BAI2) for reconciliation.
- `POST /treasury/reconcile`: Match bank transactions with internal GL journal entries using the AI-assisted matching engine.
- `GET /treasury/forecast`: Generate a cash forecast based on AP/AR schedules and planned investments.
- `POST /treasury/investments`: Record a new short-term investment.
- `POST /treasury/bank/sync`: Trigger real-time synchronization with embedded banking APIs (e.g., Bank of America).
- `GET /treasury/liquidity-dashboard`: View real-time liquidity trends and cash shortfall alerts.

## Inter-service Interaction

- Queries `Financials` for AP (Payables) and AR (Receivables) schedules to feed cash forecasting.
- Receives `Payment_Made` and `Payment_Received` events from `Financials` to update internal cash records.
- Emits `Cash_Shortfall_Alert` to management when forecasted cash drops below a threshold.
- Emits `Bank_Fees_Posted` to `Financials` for ledger recording.
- Queries `GTM (Global Trade Management)` for currency risk and trade settlement details.

## Key Logic

- **Auto-Reconciliation:** Rule-based matching of bank statement lines to GL transactions using the "Record-to-Report Assurance Advisor" to identify anomalies.
- **Liquidity Management:** Sweeping or pooling cash from subsidiary accounts into a master concentration account (In-House Banking).
- **Foreign Exchange (FX) Risk:** Tracking exposure to currency fluctuations and suggesting hedging strategies based on AI market analysis.
- **Embedded Banking:** Real-time API integration with financial institutions to initiate payments and retrieve balances without file-based exchange.
- **Cash Basis Accounting:** Selective cash basis reporting for payables and receivables cash events.
