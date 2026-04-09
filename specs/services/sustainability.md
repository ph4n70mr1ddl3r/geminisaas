# Sustainability Service Specification

## Overview

The Sustainability service enables organizations to track their environmental, social, and governance (ESG) footprint. This aligns with Oracle Fusion Cloud Sustainability.

## Data Model (SQLite)

- `esg_metrics`: id (UUID), category (CARBON/WATER/WASTE/SOCIAL), name, unit_of_measure, target_value, period_id.
- `emission_sources`: id (UUID), facility_id, fuel_type (ELECTRICITY/GAS/DIESEL), activity_data (DECIMAL), emission_factor (DECIMAL).
- `ghg_inventory`: id (UUID), emission_source_id, scope (1/2/3), co2_equivalent, date.
- `supplier_sustainability`: id (UUID), vendor_id (link to Procurement), esg_rating, status (QUALIFIED/PROBATION/REJECTED).
- `sustainability_goals`: id (UUID), name, status (ON_TRACK/AT_RISK/MET), end_date, progress_pct.

## API Endpoints (Axum/gRPC)

- `POST /sustainability/ingest-activity`: Record activity data (e.g., fuel consumption) to calculate emissions.
- `GET /sustainability/emissions-report`: Generate a Greenhouse Gas (GHG) inventory.
- `GET /sustainability/supplier-esg/{vendor_id}`: Retrieve a supplier's sustainability profile.
- `POST /sustainability/goals`: Create and track ESG targets.
- `GET /sustainability/footprint/facility/{id}`: View environmental impact for a specific site.

## Inter-service Interaction

- Receives `Receipt_Recorded` from `Procurement` to capture scope 3 emissions (purchased goods).
- Receives `Fuel_Consumption_Recorded` from `Maintenance/Logistics` for scope 1 emissions.
- Emits `ESG_Alert` to `Financials` for potential regulatory penalties or reporting requirements.
- Queries `Inventory Service` for product lifecycle data to determine environmental impact of items.

## Key Logic

- **Carbon Footprint Calculation:** Converting activity data (liters of fuel, kWh of electricity) into CO2 equivalents using standard emission factors (e.g., IPCC, EPA).
- **Scope Categorization:** Automatically classifying emissions into Scope 1 (direct), Scope 2 (indirect energy), and Scope 3 (value chain).
- **Supply Chain Decarbonization:** Assessing suppliers based on their sustainability ratings and prioritizing low-carbon options.
- **Goal Tracking:** Automated monitoring of progress toward "Net Zero" or other environmental targets.
- **Reporting Frameworks:** Mapping collected data to international standards like GRI, SASB, or TCFD.
