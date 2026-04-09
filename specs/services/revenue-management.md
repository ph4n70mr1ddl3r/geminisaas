# Revenue Management Service Specification

## Overview

The Revenue Management service (RMCS) handles revenue recognition in compliance with ASC 606 and IFRS 15. It manages performance obligations, contract liabilities, and revenue allocation across multi-element arrangements.

## Data Model (SQLite)

- `revenue_contracts`: id (UUID), name, source_order_id (link to Order Management), customer_id, total_transaction_price, status (ACTIVE/CLOSED), start_date, end_date.
- `performance_obligations` (POB): id (UUID), contract_id, name, standalone_selling_price (SSP), allocated_price, satisfaction_method (POINT_IN_TIME/OVER_TIME), percentage_complete.
- `revenue_schedules`: id (UUID), pob_id, period_id (link to Financials), amount, status (UNRECOGNIZED/RECOGNIZED).
- `contract_liabilities`: id (UUID), contract_id, balance, type (UNEARNED_REVENUE/DEFERRED_REVENUE).
- `standalone_selling_prices`: id (UUID), item_id, price, effective_date, method (LIST_PRICE/ADJUSTED_MARKET/RESIDUAL).

## API Endpoints (Axum/gRPC)

- `POST /revenue/contracts`: Ingest a sales order to create a revenue contract.
- `POST /revenue/pob/allocate`: Automatically allocate transaction price across performance obligations based on SSP.
- `POST /revenue/recognize`: Trigger revenue recognition for a specific period based on satisfaction criteria.
- `GET /revenue/reports/waterfall`: Generate a revenue waterfall report showing recognized and deferred revenue.
- `GET /revenue/contracts/{id}/liabilities`: View current contract liability balances.

## Inter-service Interaction

- Receives `Order_Placed` from `Order Management` to initiate contract creation.
- Receives `Shipment_Delivered` from `Logistics` to trigger point-in-time revenue recognition.
- Receives `Project_Milestone_Reached` from `Project Management` for percentage-of-completion recognition.
- Emits `Revenue_Recognized` to `Financials` to create GL journal entries (Debit: Deferred Revenue, Credit: Revenue).

## Key Logic

- **Transaction Price Allocation:** Implementing the 5-step model of ASC 606, specifically Step 4 (allocating the transaction price).
- **SSP Management:** Calculating Standalone Selling Prices using various estimation methods.
- **Contract Modifications:** Handling changes to existing contracts (e.g., upsells, downsells) and their impact on future revenue.
- **Variable Consideration:** Estimating and adjusting revenue based on discounts, rebates, or performance bonuses.
