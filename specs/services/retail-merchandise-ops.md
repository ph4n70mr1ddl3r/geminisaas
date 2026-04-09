# Merchandise Operations (Retail ERP) Service Specification

## Overview

The Merchandise Operations service serves as the core foundational ERP for retail, managing the item master, organizational hierarchy, costs, prices, and stock ledgers specific to retail environments. This aligns with Oracle Retail Merchandising System (RMS).

## Data Model (SQLite)

- `retail_organizational_hierarchy`: id (UUID), type (COMPANY/CHAIN/REGION/DISTRICT/STORE), name, parent_id.
- `retail_merchandise_hierarchy`: id (UUID), type (DIV/DEPT/CLASS/SUBCLASS), name, parent_id.
- `retail_item_master`: id (UUID), item_number, description, subclass_id, uom, status (ACTIVE/DISCONTINUED/PHASE_OUT).
- `retail_stock_ledger`: id (UUID), location_id, item_id, period_id, opening_stock_value, purchases_value, sales_value, closing_stock_value.
- `retail_cost_mgmt`: id (UUID), item_id, location_id, base_cost, net_cost, estimated_landed_cost.
- `retail_price_mgmt`: id (UUID), item_id, location_id, current_retail_price, clearance_price, status.

## API Endpoints (Axum/gRPC)

- `POST /retail-ops/items`: Create or update a retail item with its full hierarchy and attributes.
- `GET /retail-ops/stock-ledger/{location_id}`: Retrieve the current stock ledger state for a store or warehouse.
- `POST /retail-ops/cost/update`: Record a cost change for an item-location.
- `POST /retail-ops/price/event`: Trigger a price change or markdown event across a group of stores.
- `GET /retail-ops/org-hierarchy`: Retrieve the full retail organizational structure.
- `POST /retail-ops/stock-count/reconcile`: Finalize a physical stock count and update the stock ledger.

## Inter-service Interaction

- Emits `Item_Master_Update` to `Commerce` and `Assortment Planning`.
- Emits `Price_Change_Triggered` to `Retail Management` (POS).
- Receives `Daily_Sales_Summary` from `Retail Management` to update the stock ledger.
- Queries `Financials` (GL) to interface total stock value and COGS.
- Emits `Cost_Variance_Alert` to buyers.

## Key Logic

- **Retail Accounting Method:** Automatically calculating stock value at retail price and maintaining the cost-to-retail ratio.
- **Stock Ledger Automation:** Orchestrating the daily recording of purchases, sales, markdowns, and transfers.
- **Price Management:** Synchronizing price changes across thousands of store locations simultaneously.
- **Organizational Scale:** Designed to handle millions of item-location intersections with high performance.
- **Discontinuance Logic:** Managing the transition of an item from "Active" to "Clearance" to "Deleted."
- **Internal Profit Elimination:** Managing the accounting impact of stock movements between different legal entities in the retail chain.
