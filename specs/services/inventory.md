# Inventory Service Specification

## Overview

The Inventory service manages items, stock levels across locations, and warehouse transactions.

## Data Model (SQLite)

- `items`: id (UUID), name, sku, description, unit_of_measure.
- `locations`: id (UUID), warehouse_name, shelf/bin_code.
- `stock_levels`: id (UUID), item_id, location_id, quantity_on_hand, quantity_reserved.
- `stock_transactions`: id (UUID), item_id, location_id, type (RECEIVE/SHIP/ADJUST), quantity, date.
- `warehouses`: id (UUID), name, address.

## API Endpoints (Axum/gRPC)

- `GET /inventory/items`: Search/View items.
- `GET /inventory/levels`: Check stock levels for an item.
- `POST /inventory/adjust`: Manually adjust stock levels.
- `GET /inventory/locations`: Manage warehouse locations.

## Inter-service Interaction

- Receives `Receipt_Recorded` from `Procurement` to increase stock.
- Receives `Order_Placed` from `Sales` (not yet defined) to reserve stock.
- Receives `Order_Shipped` from `Sales` to decrease stock.
- Emits `Stock_Low` events when levels fall below a threshold.

## Key Logic

- **Reservation System:** Quantities can be reserved for orders before they are physically shipped.
- **Valuation:** FIFO (First-In, First-Out) or Weighted Average Cost (WAC) valuation methods.
- **Serial/Lot Tracking:** Capability to track individual units or batches (optional but recommended).
