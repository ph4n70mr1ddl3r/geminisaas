# Food & Beverage Management (Simphony) Service Specification

## Overview

The Food & Beverage Management service provides Point-of-Sale (POS) and kitchen management capabilities for restaurants, bars, and cafes. This aligns with Oracle Simphony.

## Data Model (SQLite)

- `revenue_centers`: id (UUID), name, type (RESTAURANT/BAR/CAFE), status, property_id (optional link to Hospitality).
- `menu_items`: id (UUID), name, category, base_price, tax_rate, allergen_info (JSON), availability_status.
- `restaurant_orders`: id (UUID), revenue_center_id, table_number, server_id, status (OPEN/FIRED/SERVED/PAID), total_amount.
- `order_line_items`: id (UUID), order_id, item_id, quantity, modifiers (JSON - e.g., 'No Onions'), status.
- `kitchen_display_tasks`: id (UUID), order_id, item_id, station_id (GRILL/SALAD/PASTA), time_fired, status (PENDING/PREPARING/READY).
- `fnb_inventory_consumption`: id (UUID), item_id, quantity_used, unit_of_measure, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /fnb/orders`: Create a new restaurant order.
- `POST /fnb/orders/{id}/fire`: Send order items to the kitchen display system (KDS).
- `POST /fnb/orders/{id}/settle`: Process payment for an order and close it.
- `GET /fnb/menu`: Retrieve the current active menu for a revenue center.
- `GET /fnb/kitchen/tasks`: Retrieve pending prep tasks for a specific kitchen station.
- `POST /fnb/inventory/waste`: Record wasted or spoiled inventory.

## Inter-service Interaction

- Queries `Hospitality Management` to post restaurant charges directly to a guest's room folio.
- Emits `Inventory_Consumed` event to `Inventory Service` for raw ingredient tracking.
- Emits `Sales_Summary_Daily` to `Financials` for revenue and tax reporting.
- Queries `Loyalty Service` to apply discounts or reward points.
- Receives `Mobile_Order_Placed` from `Commerce` for off-premises dining.

## Key Logic

- **Kitchen Workflow Optimization:** Routing items to different kitchen stations (e.g., steaks to Grill, salads to Pantry) based on item type.
- **Modifier Logic:** Handling complex customizations (e.g., "Medium Rare", "Sub Fries for Salad") and ensuring the price is adjusted correctly.
- **Table Management:** Tracking the status of physical tables (Vacant, Occupied, Dirty) and guest turn-around time.
- **Split Check Logic:** Allowing a single order to be split into multiple payments by item or amount.
- **Labor Cost Tracking:** Correlating sales volume with clocked-in staff hours to analyze labor productivity.
- **Offline POS Resilience:** Enabling orders and payments to be processed even during internet outages, with auto-sync when restored.
