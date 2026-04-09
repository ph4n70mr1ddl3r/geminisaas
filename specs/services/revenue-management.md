# Revenue Management Service Specification

## Overview

The Revenue Management service (RMCS) handles revenue recognition in compliance with ASC 606 and IFRS 15. It manages performance obligations, contract liabilities, and revenue allocation across multi-element arrangements. It includes 2026 features such as service start date as a performance event and AI-driven revenue forecasting.

## Data Model (SQLite)

- `revenue_contracts`: id (UUID), name, source_order_id (link to Order Management), customer_id, total_transaction_price, status (ACTIVE/CLOSED), start_date, end_date.
- `performance_obligations` (POB): id (UUID), contract_id, name, standalone_selling_price (SSP), allocated_price, satisfaction_method (POINT_IN_TIME/OVER_TIME/SERVICE_START), percentage_complete.
- `revenue_schedules`: id (UUID), pob_id, period_id (link to Financials), amount, status (UNRECOGNIZED/RECOGNIZED).
- `contract_liabilities`: id (UUID), contract_id, balance, type (UNEARNED_REVENUE/DEFERRED_REVENUE).
- `standalone_selling_prices`: id (UUID), item_id, price, effective_date, method (LIST_PRICE/ADJUSTED_MARKET/RESIDUAL).
- `revenue_event_triggers`: id (UUID), pob_id, event_type (SHIPMENT/MILESTONE/SERVICE_START/BILLING), trigger_date.

## API Endpoints (Axum/gRPC)

- `POST /revenue/contracts`: Ingest a sales order to create a revenue contract.
- `POST /revenue/pob/allocate`: Automatically allocate transaction price across performance obligations using the AI allocation engine.
- `POST /revenue/recognize`: Trigger revenue recognition for a specific period based on satisfaction criteria or performance events (e.g., Service Start).
- `GET /revenue/reports/waterfall`: Generate a revenue waterfall report with 2026 AI-driven projections for future periods.
- `GET /revenue/contracts/{id}/liabilities`: View current contract liability balances.
- `POST /revenue/forecast`: Generate a predictive revenue forecast based on historical trends and current order backlog.

## Inter-service Interaction

- Receives `Order_Placed` from `Order Management` to initiate contract creation.
- Receives `Shipment_Delivered` from `Logistics` to trigger point-in-time revenue recognition.
- Receives `Project_Milestone_Reached` from `Project Management` for percentage-of-completion recognition.
- Receives `Subscription_Activated` from `Subscription Management` to use "Service Start Date" as a performance event.
- Emits `Revenue_Recognized` to `Financials` to create GL journal entries.

## Key Logic

- **Transaction Price Allocation:** Implementing the 5-step model of ASC 606, specifically Step 4 (allocating the transaction price) using automated SSP rules.
- **SSP Management:** Calculating Standalone Selling Prices using various estimation methods and historical price analysis.
- **Contract Modifications:** Handling changes to existing contracts (e.g., upsells, downsells) and their impact on future revenue (Step 3: Determine Transaction Price).
- **Service Start Recognition:** Automatically triggering recognition upon the "Service Start Date" for cloud or subscription-based services.
- **Variable Consideration:** Estimating and adjusting revenue based on AI predictions for discounts, rebates, or performance bonuses.
- **Selective Cash Basis:** Specialized recognition logic for payables and receivables cash events where required by specific jurisdictions.
