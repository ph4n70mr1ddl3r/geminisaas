# Profitability & Cost Management Service Specification

## Overview

The Profitability and Cost Management (PCM) service enables business users to build and maintain complex allocation models to identify profitable segments (products, customers, or services). This aligns with Oracle Fusion Cloud Profitability and Cost Management.

## Data Model (SQLite)

- `pcm_models`: id (UUID), name, status (DRAFT/DEPLOYED), total_cost_pool, total_revenue_pool.
- `pcm_dimensions`: id (UUID), model_id, name (PRODUCT/CUSTOMER/CHANNEL/ENTITY).
- `pcm_rules`: id (UUID), model_id, name, sequence, source_pool (JSON), destination_pool (JSON), allocation_method (FIXED/VARIABLE/PRO_RATA).
- `pcm_drivers`: id (UUID), name, type (HEADCOUNT/SQUARE_FOOTAGE/ORDER_VOLUME), value_query (JSON).
- `pcm_results`: id (UUID), model_id, period_id, dimension_member_id, allocated_cost, allocated_revenue, profit_margin.

## API Endpoints (Axum/gRPC)

- `POST /pcm/models`: Create a new profitability model.
- `POST /pcm/rules`: Define an allocation rule for distributing costs or revenue.
- `POST /pcm/calculate`: Execute the allocation engine for a specific period.
- `GET /pcm/results/profitability-report`: Retrieve the final profitability analysis.
- `GET /pcm/scenarios/what-if`: Run a simulation based on modified cost or driver assumptions.

## Inter-service Interaction

- Queries `Financials Service` for real-time cost and revenue balances.
- Queries `EPM Service` (Planning) for budget and forecast data to compare against actual profitability.
- Emits `Allocation_Completed` event for audit and reporting.
- Queries `Inventory/Commerce` for volume-based drivers (e.g., units sold).

## Key Logic

- **Step-Down Allocations:** Distributing costs from shared service departments (e.g., IT, HR) to operating departments, then to final products.
- **Driver-Based Costing:** Using operational metrics (e.g., number of support tickets) to assign costs to specific customers.
- **Traceability:** Providing a "Value Flow" audit trail that shows exactly how a cost was allocated from source to destination.
- **What-If Simulations:** Creating temporary scenarios to see the impact of increasing a cost driver or reducing a budget.
- **Profitability Curves:** Generating visual reports that rank products or customers from most to least profitable.
