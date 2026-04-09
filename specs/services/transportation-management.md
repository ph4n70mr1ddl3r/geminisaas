# Transportation Management (OTM) Service Specification

## Overview

The Transportation Management (OTM) service optimizes the movement of goods, managing freight planning, execution, and settlement across all modes of transport. This aligns with Oracle Fusion Cloud Transportation Management.

## Data Model (SQLite)

- `transportation_orders`: id (UUID), source_order_id (link to Order Mgmt/Procurement), status (UNPLANNED/PLANNED/IN_TRANSIT/DELIVERED), priority.
- `shipment_plans`: id (UUID), name, status (DRAFT/EXECUTING), carrier_id, route_id, total_cost, total_weight, total_volume.
- `shipment_stops`: id (UUID), shipment_plan_id, location_id, sequence, arrival_date, departure_date, stop_type (PICKUP/DROP).
- `carrier_rates`: id (UUID), carrier_id, mode (GROUND/AIR/OCEAN), rate_type (FLAT/BY_WEIGHT), amount, effective_date.
- `freight_settlements`: id (UUID), shipment_plan_id, invoice_number, status (PENDING/APPROVED/PAID), amount.
- `transportation_track_events`: id (UUID), shipment_id, location, status_code (EN_ROUTE/DELAYED/DELIVERED), timestamp.

## API Endpoints (Axum/gRPC)

- `POST /otm/orders`: Ingest a new transportation requirement.
- `POST /otm/plan`: Execute the planning engine to create optimal shipments and routes.
- `GET /otm/shipments/{id}`: View real-time status and tracking for a shipment.
- `POST /otm/settlement/approve`: Approve a carrier invoice for payment.
- `GET /otm/carrier-performance`: Retrieve metrics on carrier reliability and cost.
- `POST /otm/event/record`: Record a tracking event from an external carrier API or IoT device.

## Inter-service Interaction

- Receives `Shipment_Required` from `Order Management` or `Procurement`.
- Emits `Shipment_Planned` event to `Logistics/WMS` to trigger picking.
- Emits `Freight_Invoice_Approved` to `Financials` (AP) for payment.
- Queries `GTM (Global Trade Management)` for trade compliance and customs documentation.
- Emits `Order_Status_Updated` to `CRM` for customer visibility.

## Key Logic

- **Route Optimization:** Using AI to find the most cost-effective and time-efficient path, considering traffic and carrier availability.
- **LTL/FTL Consolidation:** Automatically grouping "Less Than Truckload" orders into "Full Truckload" shipments to save costs.
- **Freight Tendering:** Managing the process of offering shipments to carriers and receiving bids or acceptances.
- **Voucher Matching:** Automatically matching carrier invoices against planned freight costs before payment.
- **Real-Time Tracking:** Providing automated updates to stakeholders based on GPS/IoT data from the carrier.
- **Sustainability:** Calculating the carbon footprint of transportation options to support ESG goals.
