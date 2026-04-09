# Supply Chain Financial Orchestration (SFO) Service Specification

## Overview

The Supply Chain Financial Orchestration (SFO) service manages the complex financial relationships and accounting flows between different business units and legal entities during supply chain transactions. This aligns with Oracle Fusion Cloud Supply Chain Financial Orchestration.

## Data Model (SQLite)

- `financial_orchestration_flows`: id (UUID), flow_name, source_bu_id, target_bu_id, orchestration_type (BUY_SELL/TRANSFER), status.
- `financial_routes`: id (UUID), flow_id, selling_unit_id, buying_unit_id, tax_regime_code, trade_agreement_id.
- `orchestration_tasks`: id (UUID), flow_id, event_type (SHIPMENT/RECEIPT/INVOICE), status (PENDING/COMPLETED), execution_date.
- `intercompany_accruals`: id (UUID), transaction_id, accrual_account_id, amount, currency, status (ACCRUED/SETTLED).
- `trade_distributions`: id (UUID), orchestration_id, gl_account_id, amount, debit_credit_flag.
- `profit_center_mapping`: id (UUID), inventory_org_id, profit_center_bu_id, effective_date.

## API Endpoints (Axum/gRPC)

- `POST /sfo/flows`: Define a new financial orchestration flow between two business units.
- `POST /sfo/process-event`: Trigger financial orchestration logic based on a supply chain event (e.g., physical receipt).
- `GET /sfo/intercompany/accruals`: Retrieve pending intercompany accruals for reconciliation.
- `POST /sfo/calculate-transfer-price`: Determine the internal transfer price based on pre-defined rules (COST_PLUS/MARKUP).
- `GET /sfo/flow-status/{tx_id}`: View the financial execution status of a physical supply chain transaction.

## Inter-service Interaction

- Receives `Shipment_Confirmed` from `Logistics/WMS` to initiate selling-side accounting.
- Receives `Receipt_Confirmed` from `Procurement/Inventory` to initiate buying-side accounting.
- Emits `Intercompany_Invoice_Required` event to `Financials` (AP/AR) to generate internal invoices.
- Queries `Tax Service` for intercompany tax and VAT requirements.
- Queries `Enterprise Data Management (EDM)` for legal entity and business unit hierarchies.

## Key Logic

- **Separation of Physical and Financial Flows:** Allowing goods to move directly from Vendor to Site, while the financial path goes through a "Procurement Hub" or "Global Distribution Center."
- **Internal Transfer Pricing:** Automatically calculating the price at which one internal unit "sells" to another based on cost, markup, or market price.
- **Intercompany Accruals:** Automatically creating accounting entries to track "uninvoiced receipts" between internal units.
- **Buy-Sell Orchestration:** Managing the complex accounting when one subsidiary acts as a buyer for another.
- **Profit Center Management:** Ensuring that the financial impact of a supply chain transaction is recorded in the correct profit center, regardless of the physical inventory location.
- **Tax Compliance:** Automatically identifying and applying the correct VAT/GST rules for cross-border internal transfers.
