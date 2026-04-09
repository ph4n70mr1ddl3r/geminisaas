# Warehouse Management (WMS) Service Specification

## Overview

The Warehouse Management (WMS) service manages high-volume warehouse operations, including inbound receiving, putaway, outbound picking, packing, and wave management. This aligns with Oracle Fusion Cloud Warehouse Management.

## Data Model (SQLite)

- `wms_locations`: id (UUID), warehouse_id (link to Logistics), zone_id, bin_id, type (STORAGE/PICK/DOCK/PACK), status (AVAIL/LOCKED/FULL).
- `inbound_shipments`: id (UUID), vendor_id, po_id, asn_number, status (EXPECTED/IN_TRANSIT/RECEIVING/COMPLETED).
- `inventory_lpn`: id (UUID), lpn_number (License Plate Number), location_id, item_id, quantity, status (ALLOCATED/PICKED/PACKED).
- `picking_tasks`: id (UUID), wave_id, shipment_id, item_id, source_location_id, quantity, user_id, status (PENDING/PICKED).
- `wms_waves`: id (UUID), name, status (DRAFT/RELEASED/PICKING/PACKED), total_orders, priority.
- `cycle_counts`: id (UUID), location_id, item_id, expected_qty, actual_qty, status (PENDING/APPROVED).

## API Endpoints (Axum/gRPC)

- `POST /wms/waves/release`: Create and release a wave of orders for picking.
- `POST /wms/receive`: Record receipt of an inbound shipment into an LPN.
- `GET /wms/putaway-suggestion`: Retrieve the optimal bin location for a received item.
- `GET /wms/picking-tasks/{user_id}`: Retrieve current picking tasks for a warehouse operator.
- `POST /wms/pack`: Record items into a shipping container and generate labels.
- `GET /wms/inventory-health`: Analyze slow-moving or aging stock.

## Inter-service Interaction

- Receives `PO_Sent` from `Procurement` to create expected inbound shipments.
- Queries `Inventory Service` for master item data and reservation status.
- Emits `Shipment_Picked` and `Shipment_Packed` to `Logistics`.
- Emits `Inventory_Adjustment` to `Financials` upon cycle count approval.
- Queries `Maintenance Service` for warehouse equipment availability (e.g., forklifts).

## Key Logic

- **Wave Planning:** Grouping orders by carrier, priority, or destination to maximize picking efficiency.
- **Directed Putaway:** Using logic based on item velocity, weight, and volume to suggest the best storage location.
- **LPN Tracking:** Managing inventory at the container level (License Plate Number) for rapid movement and scanning.
- **Cross-Docking:** Directly moving received items from the inbound dock to an outbound shipment without putaway.
- **Task Interleaving:** Dynamically assigning the next best task to a worker (e.g., a putaway near their current location).
- **Cluster Picking:** Allowing a single worker to pick multiple orders simultaneously into separate bins on a cart.
