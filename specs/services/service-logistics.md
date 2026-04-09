# Service Logistics Service Specification

## Overview

The Service Logistics service bridges the gap between Field Service and Supply Chain, managing the parts required for service activities, including part orders, returns, and depot repair. This aligns with Oracle Fusion Cloud Service Logistics.

## Data Model (SQLite)

- `parts_requirements`: id (UUID), service_activity_id (link to Field Service), item_id (link to Inventory), quantity, status (PENDING/ORDERED/SHIPPED/RECEIVED/INSTALLED).
- `part_orders`: id (UUID), requirement_id, order_number, carrier_id, tracking_number.
- `part_returns`: id (UUID), original_requirement_id, reason_code, status (REQUESTED/IN_TRANSIT/RECEIVED).
- `depot_repairs`: id (UUID), asset_id, repair_type, status (ARRIVED/DIAGNOSIS/REPAIRING/FINISHED/SHIPPED_BACK).
- `technician_trunk_stock`: id (UUID), technician_id, item_id, quantity.
- `service_debits`: id (UUID), activity_id, part_cost, labor_cost, total_charge, status (PENDING/BILLED).

## API Endpoints (Axum/gRPC)

- `POST /service-logistics/require-parts`: Submit a parts requirement for a field service task.
- `GET /service-logistics/technician-stock`: View current stock in a technician's vehicle.
- `POST /service-logistics/returns`: Create a return authorization for defective parts.
- `GET /service-logistics/repair-status`: Track an asset sent for depot repair.
- `POST /service-logistics/post-charges`: Submit final service costs for billing.

## Inter-service Interaction

- Receives `Activity_Scheduled` from `Field Service` to trigger part procurement.
- Queries `Inventory Service` for stock availability in local warehouses.
- Emits `Part_Ordered` event to `Logistics` to initiate shipping to the technician's location.
- Emits `Service_Charge_Posted` to `Project Management/Financials` to generate an invoice.
- Queries `Fixed Assets` for asset warranty information to determine if a repair is billable.

## Key Logic

- **Automated Replenishment:** Triggering new orders when a technician's trunk stock falls below a certain level.
- **Warranty Verification:** Checking asset history to automatically determine if parts and labor should be billed or covered.
- **Cross-Docking for Service:** Coordinating parts delivery to arrive exactly when and where the technician is scheduled.
- **RMA (Return Material Authorization):** Managing the lifecycle of defective parts from field collection to warehouse receipt.
- **Depot Cycle Time:** Tracking and optimizing the time it takes to repair and return a customer asset.
