# Maintenance Service Specification

## Overview

The Maintenance service manages the health and performance of internal and external assets (machinery, vehicles, facilities). It handles preventive and corrective maintenance.

## Data Model (SQLite)

- `assets`: id (UUID), name, type (MACHINE/VEHICLE/FACILITY), status (OPERATIONAL/DOWN/UNDER_REPAIR), serial_number, location.
- `maintenance_plans`: id (UUID), asset_id, frequency (DAYS/USAGE_HOURS), last_maintenance_date.
- `maintenance_work_orders`: id (UUID), asset_id, type (PREVENTIVE/CORRECTIVE), status (OPEN/IN_PROGRESS/COMPLETED/CLOSED), description, technician_id.
- `technicians`: id (UUID), name, skill_set, hourly_rate (DECIMAL).
- `maintenance_parts`: id (UUID), work_order_id, item_id (link to Inventory), quantity_used.
- `asset_readings`: id (UUID), asset_id, reading_type (TEMP/USAGE_HOURS), reading_value, reading_date.

## API Endpoints (Axum/gRPC)

- `POST /maintenance/assets`: Create and track enterprise assets.
- `GET /maintenance/plans`: View/Set preventive maintenance schedules.
- `POST /maintenance/work-orders`: Generate work orders for asset repair or service.
- `POST /maintenance/readings`: Record telemetry data (IoT integration) for predictive maintenance.
- `GET /maintenance/asset-history`: View full maintenance history for an asset.

## Inter-service Interaction

- Queries `Inventory Service` for spare parts availability.
- Emits `Part_Used` event to `Inventory` when parts are used in maintenance.
- Emits `Asset_Down` event to `Manufacturing` to alert of machine unavailability.
- Emits `Maintenance_Cost` event to `Financials` for asset capitalization or expense tracking.

## Key Logic

- **Preventive Scheduling:** Automatically generate work orders based on time or usage thresholds.
- **Predictive Analytics:** Use asset readings (IoT) to predict failures before they occur.
- **Availability Management:** Track "mean time between failures" (MTBF) and "mean time to repair" (MTTR).
- **Resource Allocation:** Assign technicians based on skills and availability.
