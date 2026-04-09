# Retail Management Service Specification

## Overview

The Retail Management service provides specialized capabilities for high-volume retail operations, including store management, merchandising, and omni-channel fulfillment. This aligns with Oracle Retail (formerly Retek).

## Data Model (SQLite)

- `retail_stores`: id (UUID), name, type (BRICK_AND_MORTAR/E-COMMERCE/KIOSK), status, location_id, manager_id.
- `merchandise_hierarchy`: id (UUID), department, class, subclass, item_id.
- `store_inventory`: id (UUID), store_id, item_id, quantity_on_hand, quantity_reserved, safety_stock.
- `retail_promotions`: id (UUID), name, store_id, start_date, end_date, discount_rules (JSON).
- `retail_sales_daily`: id (UUID), store_id, date, total_sales, total_tax, total_margin.
- `omni_channel_orders`: id (UUID), source (WEB/APP/STORE), type (BOPIS/BORIS/SHIP_FROM_STORE), status.

## API Endpoints (Axum/gRPC)

- `GET /retail/store/inventory`: Retrieve real-time stock levels for a specific store.
- `POST /retail/inventory/transfer`: Move stock between stores to balance demand.
- `POST /retail/promotions/activate`: Launch a new price promotion across a subset of stores.
- `POST /retail/sales/upload`: Ingest daily sales totals from Point-of-Sale (POS) systems.
- `GET /retail/fulfillment/options`: Determine the best fulfillment method (e.g., "Pick up in Store") for a customer.
- `GET /retail/analytics/sell-through`: Analyze how quickly items are selling at different price points.

## Inter-service Interaction

- Queries `Inventory Service` for master item data and distribution center levels.
- Receives `Order_Placed` from `Commerce` to trigger omni-channel fulfillment.
- Emits `Sales_Summary` event to `Financials` (GL) for revenue recording.
- Queries `Loyalty Service` for customer-specific retail offers.
- Emits `Stockout_Alert` to `Supply Chain Planning` for urgent replenishment.

## Key Logic

- **BOPIS (Buy Online, Pick Up In Store):** Managing the workflow for store associates to pick and hold web orders for customer pickup.
- **Price Markdown Optimization:** Using AI to suggest the best time and amount for markdowns to clear seasonal inventory.
- **Store-to-Store Transfers:** Identifying overstocked and understocked stores and recommending optimal stock movements.
- **Planogram Alignment:** Ensuring that store inventory levels match the physical shelf space (planogram) constraints.
- **Fraudulent Return Detection:** Analyzing return patterns across stores to identify potential retail fraud.
- **Nightly POS Polling:** High-scale ingestion of millions of store transactions for daily financial reconciliation.
