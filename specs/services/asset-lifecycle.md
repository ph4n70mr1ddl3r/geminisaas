# Asset Lifecycle Management (ALM) Service Specification

## Overview

The Asset Lifecycle Management (ALM) service provides a unified view of organizational assets, combining financial asset tracking (Fixed Assets) with operational maintenance. This aligns with Oracle Fusion Cloud Asset Management and Maintenance.

## Data Model (SQLite)

- `alm_assets`: id (UUID), name, status (DRAFT/ACTIVE/MAINTENANCE/RETIRED), serial_number, current_location, financial_asset_id (link to Fixed Assets), maintenance_plan_id.
- `alm_meters`: id (UUID), asset_id, meter_type (HOURS/KILOMETERS/CYCLES), current_reading, date_recorded.
- `alm_service_history`: id (UUID), asset_id, type (PREVENTIVE/CORRECTIVE), work_order_id, cost, date.
- `alm_depreciation_forecasts`: id (UUID), asset_id, period_id, forecasted_value, status.
- `alm_asset_relationships`: id (UUID), parent_asset_id, child_asset_id, relationship_type (SUBASSEMBLY/COMPONENT).
- `alm_asset_warranties`: id (UUID), asset_id, vendor_id, expiry_date, status.

## API Endpoints (Axum/gRPC)

- `POST /alm/assets`: Create or register a new physical asset.
- `GET /alm/assets/{id}/health`: Retrieve asset health score based on meter readings and service history.
- `POST /alm/meters/record`: Update an asset's usage meter reading.
- `GET /alm/depreciation/forecast`: Retrieve AI-driven depreciation projections for an asset.
- `POST /alm/transfer`: Record the physical movement of an asset between locations.
- `GET /alm/asset-hierarchy`: View a tree-based view of asset subassemblies and components.

## Inter-service Interaction

- Queries `Fixed Assets Service` for financial values, depreciation, and tax context.
- Emits `Maintenance_Triggered` event to `Maintenance Service` when a meter hits a threshold.
- Receives `Work_Order_Completed` from `Maintenance` to update asset service history.
- Emits `Asset_Location_Changed` event to `Logistics` for tracking movements.
- Queries `GTM (Global Trade Management)` for assets being moved across borders.
- Emits `Asset_Anomaly_Detected` to `Risk Management` for potential theft or misuse.

## Key Logic

- **Unified Asset 360:** Combining financial and operational data to show the true cost of ownership (TCO) for an asset.
- **Predictive Maintenance:** Using AI to analyze meter readings and service trends to predict failures before they occur.
- **Genealogy & Hierarchy:** Tracking complex parent-child relationships for assets like aircraft or heavy machinery.
- **Capitalization Lifecycle:** Automatically identifying when an operational maintenance cost should be capitalized into the asset's basis.
- **Meter-Based Depreciation:** Optionally calculating asset depreciation based on actual usage (units-of-production method) rather than time.
- **Remote Monitoring:** Integrating with IoT sensors to provide real-time location and telemetry data.
