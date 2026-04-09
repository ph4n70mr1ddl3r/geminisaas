# Financials Service Specification

## Overview

The Financials service handles General Ledger, Accounts Payable, Accounts Receivable, and Cash Management.

## Data Model (SQLite)

- `ledgers`: id (UUID), name, currency (USD/EUR).
- `chart_of_accounts`: id (UUID), account_code (e.g., '10100'), account_name ('Cash'), account_type ('Asset').
- `journal_entries`: id (UUID), date, description, status (DRAFT/POSTED).
- `journal_lines`: id (UUID), journal_entry_id, account_id, debit (DECIMAL), credit (DECIMAL).
- `periods`: id (UUID), start_date, end_date, closed (BOOL).
- `invoices`: id (UUID), type (PAYABLE/RECEIVABLE), vendor_id (link to Procurement), amount, status.

## API Endpoints (Axum/gRPC)

- `POST /gl/journals`: Create/post a journal entry.
- `GET /gl/balances`: Get current account balances for a period.
- `POST /ap/invoices`: Create a payable invoice.
- `GET /ar/invoices`: List receivable invoices.
- `GET /gl/reports/balance-sheet`: Generate balance sheet report.
- `GET /gl/reports/income-statement`: Generate income statement report.

## Inter-service Interaction

- Receives events from Procurement when an invoice is matched to a PO.
- Receives events from Sales/Inventory when an order is fulfilled.
- Emits events when a payment is made or received.
- Calls `Identity Service` to verify permissions for sensitive financial operations.

## Key Logic

- **Double-Entry Bookkeeping:** Every journal entry must have debits equal to credits.
- **Period Control:** No postings allowed in closed periods.
- **Audit Logging:** Every change must be tracked with user and timestamp.
