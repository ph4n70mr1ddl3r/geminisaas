# Logistics Service Specification

## Overview

The Logistics service handles Warehouse Management (WMS), Transportation Management (OTM), and Global Trade Management (GTM). It manages the movement of goods from warehouses to customers.

## Data Model (SQLite)

- `warehouses`: id (UUID), name, location_code, address, total_capacity.
- `inventory_bins`: id (UUID), warehouse_id, bin_id, location (AISLE/SHELF/LEVEL).
- `shipments`: id (UUID), order_id (link to CRM), status (PENDING/PICKED/PACKED/SHIPPED/DELIVERED), carrier_id, tracking_number.
- `shipment_lines`: id (UUID), shipment_id, item_id (link to Inventory), quantity, weight, volume.
- `carriers`: id (UUID), name, service_level (GROUND/AIR/OCEAN), base_rate (DECIMAL).
- `freight_costs`: id (UUID), shipment_id, freight_amount, fuel_surcharge, tax.
- `trade_compliance_checks`: id (UUID), shipment_id, status (PASSED/FAILED), check_type (GTM/SANCTION/RESTRICTION).

## API Endpoints (Axum/gRPC)

- `POST /logistics/shipment`: Create and track shipments.
- `GET /logistics/warehouse-map`: View real-time inventory bin availability.
- `POST /logistics/pick-pack`: Record picking and packing of shipment items.
- `GET /logistics/freight-quote`: Calculate estimated freight costs based on weight and distance.
- `POST /logistics/compliance-check`: Perform global trade compliance check.

## Inter-service Interaction

- Receives `Order_Fulfilled` event from `CRM/Sales` to initiate shipping process.
- Emits `Shipment_Shipped` event to `CRM` to update order status.
- Queries `Inventory Service` for stock reservation and bin location.
- Emits `Shipment_Cost` event to `Financials` for freight expense recording.

## Key Logic

- **Wave Picking:** Group multiple shipments together for efficient warehouse picking.
- **Dynamic Routing:** Select carriers based on cost, speed, and reliability.
- **LTL/FTL Consolidation:** Group shipments (Less than Truckload) into Full Truckloads to save costs.
- **Cross-Docking:** Direct transfer of received goods from receiving dock to shipping dock with minimal storage.
