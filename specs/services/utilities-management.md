# Utilities Management Service Specification

## Overview

The Utilities Management service provides specialized capabilities for managing utility infrastructure, including electricity, gas, and water assets. It handles complex service histories, field crew operations, and integration with GIS. This aligns with Oracle Utilities Work and Asset Cloud Service (WACS).

## Data Model (SQLite)

- `utility_assets`: id (UUID), name, type (POLE/TRANSFORMER/PIPE/METER), status (INSTALLED/REPAIR/INACTIVE), location_gis (JSON), financial_id.
- `service_histories`: id (UUID), asset_id, event_type (INSPECTION/REPAIR/INSTALL), date, findings, results (JSON).
- `field_crews`: id (UUID), name, skills (JSON), home_base, current_status (IN_FIELD/EN_ROUTE/OFF_DUTY).
- `utility_work_orders`: id (UUID), asset_id, type (EMERGENCY/PLANNED), priority, scheduled_date, crew_id, status.
- `conditional_questionnaires`: id (UUID), work_order_id, questions (JSON), answers (JSON), version.
- `asset_telemetry`: id (UUID), asset_id, reading_type (VOLTAGE/PRESSURE/FLOW), value, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /utilities/assets`: Register a new utility asset with GIS coordinates.
- `POST /utilities/work-orders`: Create and schedule a utility-specific maintenance task.
- `GET /utilities/crew/my-tasks`: Retrieve the day's assignments for a field crew.
- `POST /utilities/service-history/record`: Log findings from an in-field asset inspection.
- `GET /utilities/asset-map`: Retrieve asset and crew locations for GIS visualization.
- `POST /utilities/telemetry/ingest`: Ingest real-time telemetry data from smart meters or sensors.

## Inter-service Interaction

- Queries `Asset Lifecycle Management (ALM)` for high-level financial and lifecycle context.
- Emits `Safety_Incident_Detected` event to `Workforce Health & Safety`.
- Receives `Inventory_Update` from `WMS` for parts required in the field.
- Queries `Construction & Engineering` for large-scale network expansion projects.
- Emits `Service_Completed` notification to customers via `B2C Service`.

## Key Logic

- **GIS Integration:** Native support for storing and querying asset data based on spatial coordinates.
- **Conditional Field Forms:** Dynamically generating follow-up questions for field crews based on their initial observations (e.g., if a pipe is leaking, ask for the flow rate).
- **Outage Management:** Identifying and grouping related asset failures to detect the root cause of a service outage.
- **Vegetation Management:** Scheduling maintenance (e.g., tree trimming) based on proximity to high-voltage assets.
- **Compatible Unit Estimating:** Automatically calculating the materials, labor, and equipment needed for a task based on "Standard Units" of work.
- **Mobile Offline Support:** Allowing field crews to capture data in remote areas without internet and sync once back in range.
