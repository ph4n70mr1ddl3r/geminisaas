# EPM (Enterprise Performance Management) Service Specification

## Overview

The EPM service handles financial planning, budgeting, forecasting, and financial consolidation. This aligns with Oracle Fusion Cloud EPM.

## Data Model (SQLite)

- `budgets`: id (UUID), name, type (OPERATING/CAPITAL), start_date, end_date, currency, status (DRAFT/APPROVED).
- `budget_lines`: id (UUID), budget_id, account_id (link to Financials), period_id, amount.
- `forecasts`: id (UUID), budget_id, name, version, amount, period_id.
- `consolidations`: id (UUID), period_id, status (PENDING/COMPLETED), reporting_currency.
- `consolidation_entities`: id (UUID), consolidation_id, ledger_id (link to Financials), ownership_percentage.

## API Endpoints (Axum/gRPC)

- `POST /epm/budgets`: Create a new budget plan.
- `GET /epm/budgets/{id}/actuals-vs-budget`: Compare real GL balances with budgeted amounts.
- `POST /epm/forecasts`: Generate a forecast based on historical actuals and future assumptions.
- `POST /epm/consolidations`: Run the consolidation process for multiple legal entities.
- `GET /epm/reports/consolidated-financials`: Retrieve consolidated Balance Sheet and Income Statement.

## Inter-service Interaction

- Queries `Financials Service` for real-time GL balances and period statuses.
- Emits `Budget_Exceeded` alerts to `Procurement` and `HCM` when spend hits thresholds.
- Queries `Identity Service` for hierarchical approvals of budgets.

## Key Logic

- **Financial Consolidation:** Aggregating financial data from multiple subsidiaries, handling intercompany eliminations, and currency translations.
- **What-If Analysis:** Creating multiple versions of forecasts based on different economic scenarios.
- **Top-Down & Bottom-Up Budgeting:** Supporting both executive-level budget allocation and department-level budget requests.
- **Allocations:** Distributing shared costs (e.g., IT, Rent) across different departments based on predefined drivers (e.g., headcount, square footage).
