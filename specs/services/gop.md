# Global Order Promising (GOP) Service Specification

## Overview

The Global Order Promising (GOP) service is a high-performance engine that calculates the most efficient way to fulfill customer orders across a global supply chain, considering inventory, manufacturing capacity, and transportation constraints. This aligns with Oracle Fusion Cloud Global Order Promising.

## Data Model (SQLite)

- `promising_rules`: id (UUID), name, priority, supply_sources (JSON), lead_time_offset.
- `available_to_promise` (ATP): id (UUID), item_id, location_id, period_id, quantity_available.
- `capable_to_promise` (CTP): id (UUID), item_id, resource_id, period_id, capacity_available.
- `fulfillment_plans`: id (UUID), order_line_id (link to Order Mgmt), promised_date, ship_from_location, carrier_service_id, status (PROPOSED/COMMITTED).
- `sourcing_rules`: id (UUID), item_id, target_location_id, source_type (TRANSFER/BUY/MAKE), source_location_id, rank.

## API Endpoints (Axum/gRPC)

- `POST /gop/check-availability`: Perform a real-time availability check for a set of items and dates.
- `POST /gop/calculate-promise`: Determine the best fulfillment date and location for an order line.
- `POST /gop/commit-promise`: Lock inventory or capacity against a promised order line.
- `GET /gop/fulfillment-plan/{order_line_id}`: Retrieve the detailed promising logic for an order.
- `POST /gop/re-promise`: Re-calculate promises for backordered or delayed orders.

## Inter-service Interaction

- Receives `Order_Line_Created` from `Order Management` to trigger promising logic.
- Queries `Inventory Service` and `WMS` for real-time stock levels.
- Queries `Supply Chain Planning` for future supply (purchase orders, work orders).
- Queries `Manufacturing Service` for resource capacity (CTP).
- Emits `Order_Promised` event to `Order Management` to update order status.
- Queries `Logistics Service` for transit times and carrier availability.

## Key Logic

- **ATP (Available to Promise):** Calculating availability based on current stock minus existing commitments.
- **CTP (Capable to Promise):** Checking if manufacturing can produce the item in time if it's not in stock.
- **Profitable to Promise:** Selecting the fulfillment path that maximizes margin (considering shipping and manufacturing costs).
- **Split Shipment Logic:** Deciding whether to ship an order from multiple locations to meet the customer's delivery date.
- **Backlog Management:** Prioritizing high-value customers when supply is constrained.
- **Lead Time Calculation:** Factoring in picking, packing, and transit times for each fulfillment node.
