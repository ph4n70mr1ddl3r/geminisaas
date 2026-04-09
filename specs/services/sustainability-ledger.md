# Sustainability Ledger Service Specification

## Overview

The Sustainability Ledger service provides a dedicated sub-ledger for recording and reporting environmental, social, and governance (ESG) metrics. It automatically captures emissions data from standard ERP/SCM transactions (e.g., travel, energy, procurement). This aligns with the Oracle Cloud Sustainability (2026-2027 roadmap).

## Data Model (SQLite)

- `esg_accounts`: id (UUID), account_name (e.g., 'Scope 1 - Stationary Combustion'), metric_type (CO2E/KWH/LITERS), status.
- `sustainability_journal_headers`: id (UUID), source_transaction_id (link to AP/Exp), period_id, total_emission_value, status (DRAFT/POSTED).
- `sustainability_journal_lines`: id (UUID), header_id, esg_account_id, quantity, emission_factor_used, co2e_amount.
- `emission_factors_central`: id (UUID), category (FLIGHT/FUEL/ELECTRICITY), region, factor_value, source (IPCC/EPA/DEFRA), effective_date.
- `sustainability_targets_ledger`: id (UUID), esg_account_id, period_id, target_value, actual_value, variance.
- `supplier_esg_scores_ledger`: id (UUID), supplier_id, carbon_intensity_index, diversity_score, risk_rating.

## API Endpoints (Axum/gRPC)

- `POST /sustainability-ledger/journals`: Create a sustainability accounting entry based on a business event.
- `POST /sustainability-ledger/calculate-emissions`: Apply the correct emission factor to a raw activity value (e.g., miles flown).
- `GET /sustainability-ledger/balances`: Retrieve the cumulative ESG metrics for a legal entity or department.
- `POST /sustainability-ledger/targets`: Set and track progress toward Net Zero or other ESG goals.
- `GET /sustainability-ledger/reports/regulatory`: Generate standard ESG reports (e.g., TCFD, CSRD, GRI).
- `POST /sustainability-ledger/factors/update`: Sync the latest global emission factors from regulatory databases.

## Inter-service Interaction

- Receives `Expense_Approved` from `Expenses` to capture travel-related emissions.
- Receives `Invoice_Posted` from `Financials` (AP) to capture energy and utility-related emissions.
- Receives `Receipt_Confirmed` from `Procurement` to capture Scope 3 (Purchased Goods) impact.
- Emits `ESG_Threshold_Violated` alert to the Chief Sustainability Officer.
- Queries `Landed Cost Management` for transport-related carbon data.
- Emits `Sustainability_Data` to `Narrative Reporting` for the annual integrated report.

## Key Logic

- **Automated ESG Accounting:** Applying the same rigor to carbon data as financial data (Double-Entry for Carbon).
- **Activity-to-Emission Conversion:** Maintaining a massive library of emission factors and automatically applying them based on transaction attributes.
- **Scope 1, 2, and 3 Classification:** Automatically categorizing emissions based on the source of the transaction.
- **Supply Chain Decarbonization:** Aggregating emissions data from suppliers via the `Supplier Portal` to build a complete Scope 3 view.
- **Sustainability Planning:** Integrating with `EPM` to model the financial impact of carbon taxes or green energy transitions.
- **Audit-Grade Traceability:** Providing a clear link from a line item in an ESG report back to the original Purchase Order or Expense report.
