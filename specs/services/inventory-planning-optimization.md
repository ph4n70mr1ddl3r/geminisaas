# Inventory Planning Optimization (IPO) Service Specification

## Overview

The Inventory Planning Optimization (IPO) service provides a unified engine for replenishment, allocation, and lifecycle inventory management. It ensures that the right quantity of product is in the right location to maximize service levels while minimizing inventory investment. This aligns with Oracle Retail Inventory Planning and Optimization.

## Data Model (SQLite)

- `replenishment_policies`: id (UUID), item_id, location_id, policy_type (MIN_MAX/EOQ/DYNAMIC), service_level_target_pct.
- `inventory_allocations`: id (UUID), source_location_id, item_id, total_qty_available, allocation_rules (JSON).
- `allocation_lines`: id (UUID), allocation_id, target_location_id, qty_allocated, status (PENDING/RELEASED).
- `stock_targets`: id (UUID), item_id, location_id, period_id, safety_stock_qty, max_stock_qty.
- `replenishment_orders_planned`: id (UUID), item_id, location_id, source_type (WH/VENDOR), qty_planned, date_needed.
- `inventory_lifecycle_status`: id (UUID), item_id, status (NEW/ACTIVE/PROMO/CLEARANCE), exit_strategy (JSON).

## API Endpoints (Axum/gRPC)

- `POST /retail-ipo/calculate-replenishment`: Execute the replenishment engine to generate planned orders for stores and warehouses.
- `POST /retail-ipo/allocations`: Create a new inventory allocation from a distribution center to stores.
- `GET /retail-ipo/stock-targets`: Retrieve AI-driven safety stock and max-stock levels for an item/location.
- `POST /retail-ipo/rebalance`: Suggest stock movements between locations to resolve shortages and overstocks.
- `GET /retail-ipo/analytics/service-level`: Analyze historical service levels (in-stock %) vs targets.
- `POST /retail-ipo/release-orders`: Transition planned replenishment into formal Purchase or Transfer Orders.

## Inter-service Interaction

- Queries `Retail Demand Forecasting (RDF)` for predicted demand and confidence scores.
- Emits `Transfer_Order_Requested` event to `Inventory / WMS` for internal stock movements.
- Emits `Purchase_Order_Requested` event to `Procurement` for vendor replenishment.
- Queries `Logistics` for lead times and carrier constraints used in replenishment logic.
- Emits `Stockout_Prediction` alert to store managers.

## Key Logic

- **Dynamic Safety Stock:** Automatically adjusting safety stock levels based on demand volatility and lead-time variability.
- **Fair-Share Allocation:** Logic for distributing limited stock across multiple stores to ensure no location is completely stocked out.
- **New Item Seeding:** Using "Like-Item" history to calculate initial inventory requirements for products without sales history.
- **Seasonality Handling:** Increasing stock targets ahead of peak seasons and decreasing them for clearance.
- **Constraint-Based Replenishment:** Considering warehouse capacity and shipping frequency when generating orders.
- **Multitier Replenishment:** Managing stock across a complex hierarchy (e.g., Global DC -> Regional WH -> Store).
