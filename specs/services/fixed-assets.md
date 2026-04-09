# Fixed Assets Service Specification

## Overview

The Fixed Assets service manages the lifecycle of tangible assets (e.g., equipment, buildings) from acquisition to retirement, including depreciation calculations. This aligns with Oracle Fusion Cloud Fixed Assets.

## Data Model (SQLite)

- `assets`: id (UUID), name, serial_number, category_id, vendor_id (link to Procurement), acquisition_date, cost, status (ACTIVE/RETIRED/FULLY_DEPRECIATED).
- `asset_categories`: id (UUID), name, depreciation_method (STRAIGHT_LINE/DECLINING_BALANCE), life_in_months.
- `asset_depreciation`: id (UUID), asset_id, period_id (link to Financials), amount, accumulated_depreciation.
- `asset_retirements`: id (UUID), asset_id, date, proceeds, gain_or_loss.
- `asset_transfers`: id (UUID), asset_id, from_department_id, to_department_id, date.

## API Endpoints (Axum/gRPC)

- `POST /assets`: Record a new asset acquisition.
- `POST /assets/{id}/depreciate`: Calculate depreciation for a period.
- `POST /assets/{id}/transfer`: Record a physical or departmental move of an asset.
- `POST /assets/{id}/retire`: Record an asset retirement (sale, scrap).
- `GET /assets/reports/value`: Generate asset valuation reports.

## Inter-service Interaction

- Receives `Receipt_Recorded` from `Procurement` for high-value items to auto-create asset records.
- Emits `Depreciation_Posted` to `Financials` to create GL journal entries for depreciation expense.
- Queries `HCM Service` for department and location master data.

## Key Logic

- **Depreciation Schedules:** Automatically generating future depreciation projections based on asset cost and lifespan.
- **Mass Additions:** Processing multiple assets from a single purchase order/invoice.
- **Impairment:** Capability to write down the value of an asset if its market value falls significantly.
- **Capitalization Thresholds:** Automated logic to determine if an expense should be capitalized as an asset or expensed directly.
