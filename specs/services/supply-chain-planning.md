# Supply Chain Planning Service Specification

## Overview

The Supply Chain Planning (SCP) service handles demand forecasting, supply planning, and sales & operations planning (S&OP). This aligns with Oracle Fusion Cloud Supply Chain Planning.

## Data Model (SQLite)

- `demand_forecasts`: id (UUID), item_id, period_id, forecasted_quantity, confidence_level, version.
- `supply_plans`: id (UUID), name, status (DRAFT/RELEASED), planning_horizon_start, planning_horizon_end.
- `planned_orders`: id (UUID), plan_id, item_id, quantity, type (PURCHASE/MAKE/TRANSFER), suggested_date.
- `safety_stock_levels`: id (UUID), item_id, warehouse_id, minimum_quantity, calculation_method.
- `resource_availability`: id (UUID), work_center_id, period_id, available_hours, planned_hours.

## API Endpoints (Axum/gRPC)

- `POST /planning/forecasts/generate`: Generate a new demand forecast using historical sales data.
- `POST /planning/run-mrp`: Execute Material Requirements Planning (MRP) to generate planned orders.
- `GET /planning/pegging`: View the relationship between a specific supply (e.g., PO) and the demand it satisfies.
- `POST /planning/release-orders`: Convert planned orders into real Purchase Orders or Work Orders.
- `GET /planning/reports/excess-inventory`: Identify items where supply significantly exceeds forecasted demand.

## Inter-service Interaction

- Queries `CRM/Order Management` for historical sales and current backlogs to feed demand forecasts.
- Queries `Inventory Service` for current stock levels and lead times.
- Queries `Manufacturing Service` for production capacity and routings.
- Emits `Planned_Order_Released` to `Procurement` or `Manufacturing` to initiate execution.

## Key Logic

- **Statistical Forecasting:** Using algorithms like Exponential Smoothing or Moving Average to predict future demand.
- **Constraint-Based Planning:** Generating plans that respect resource limitations (e.g., machine capacity, labor availability).
- **Lead Time Offsetting:** Back-scheduling from the demand date to determine when an order must be placed.
- **S&OP Consensus:** Providing a collaborative environment where Sales, Finance, and Operations can agree on a single operating plan.
