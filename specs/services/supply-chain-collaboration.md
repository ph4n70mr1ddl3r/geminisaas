# Supply Chain Collaboration Service Specification

## Overview

The Supply Chain Collaboration service provides a unified platform for sharing forecasts, orders, and inventory data with trading partners to improve supply chain visibility and responsiveness. This aligns with Oracle Fusion Cloud Supply Chain Collaboration.

## Data Model (SQLite)

- `collaboration_plans`: id (UUID), name, partner_id (link to Vendor), status (ACTIVE/DRAFT), period_id.
- `order_forecasts`: id (UUID), plan_id, item_id, forecast_qty, period_id, source (OWNER/PARTNER).
- `forecast_commits`: id (UUID), forecast_id, commit_qty, status (COMMITTED/DEVIATED), comments.
- `collaboration_orders`: id (UUID), plan_id, po_number, status (RELEASED/ACKNOWLEDGED/REJECTED), delivery_date.
- `vmi_inventory`: id (UUID), item_id, location_id, current_qty, min_threshold, max_threshold, status.
- `collaboration_messages`: id (UUID), plan_id, sender_id, text, timestamp, status.

## API Endpoints (Axum/gRPC)

- `POST /collaboration/plans`: Create a new collaboration plan with a supplier.
- `POST /collaboration/forecasts/share`: Publish an order forecast to a supplier.
- `POST /collaboration/forecasts/commit`: Supplier recording their commitment to a forecast.
- `GET /collaboration/vmi/status`: Monitor Vendor Managed Inventory (VMI) levels.
- `POST /collaboration/orders/release`: Release a planned order to a supplier based on collaboration logic.
- `GET /collaboration/performance-metrics`: Track supplier commit accuracy and lead-time reliability.

## Inter-service Interaction

- Queries `Supply Chain Planning` for demand and supply forecasts.
- Emits `Forecast_Commit_Received` to `Planning` to update supply assumptions.
- Receives `Inventory_Update` from `WMS/Inventory` for VMI tracking.
- Emits `Order_Released` to `Procurement` to generate a Purchase Order.
- Queries `Supplier Portal` for supplier-specific constraints (e.g., minimum order quantities).

## Key Logic

- **Commit Tracking:** Identifying the gap between what the company forecasted and what the supplier committed to deliver.
- **Vendor Managed Inventory (VMI):** Allowing suppliers to monitor inventory levels and automatically trigger replenishments within set thresholds.
- **Multitier Collaboration:** Extending visibility to Tier 2 and Tier 3 suppliers for critical components.
- **Order Acknowledgment:** Requiring suppliers to digitally acknowledge and date-commit to all purchase orders.
- **Forecast Waterfall:** Comparing successive versions of forecasts to analyze accuracy and variability over time.
- **Social Collaboration:** Real-time messaging between planners and supplier representatives within the context of a specific order or forecast.
