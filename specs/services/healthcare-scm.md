# Healthcare Supply Chain Service Specification

## Overview

The Healthcare Supply Chain service provides specialized logistics and procurement capabilities for the healthcare industry, ensuring clinical alignment, patient safety, and regulatory compliance. This aligns with Oracle Fusion Cloud Healthcare Supply Chain.

## Data Model (SQLite)

- `clinical_items`: id (UUID), name, type (IMPLANT/DRUG/EQUIPMENT), fda_classification, storage_temp_req, udi (Unique Device Identifier).
- `care_locations`: id (UUID), facility_name, department (ER/OR/ICU), bed_id, status.
- `case_carts`: id (UUID), surgery_id (link to Project Mgmt/External), item_list (JSON), location_id, status (READY/PICKED/USED).
- `recall_notices`: id (UUID), manufacturer_id, item_id, lot_numbers (JSON), severity, status (OPEN/REMEDIATED).
- `consignment_inventory`: id (UUID), vendor_id, item_id, location_id, quantity_on_hand, status (OWNED_BY_VENDOR/PURCHASED).
- `clinical_par_levels`: id (UUID), location_id, item_id, min_qty, max_qty, replenishment_priority.

## API Endpoints (Axum/gRPC)

- `GET /healthcare-scm/case-cart/status`: Retrieve the readiness status of items for a planned surgical case.
- `POST /healthcare-scm/recall/initiate`: Trigger an internal recall search across all warehouses and clinics.
- `POST /healthcare-scm/replenish/par`: Automatically generate replenishment requests based on PAR levels.
- `GET /healthcare-scm/consignment/usage`: Report on vendor-owned inventory consumed by the clinic.
- `POST /healthcare-scm/udi/scan`: Record the use of a high-value clinical item using its UDI.
- `GET /healthcare-scm/analytics/clinically-integrated`: View cost-per-case analysis combining SCM and clinical outcomes.

## Inter-service Interaction

- Queries `Inventory Service` for physical stock levels and warehouse metadata.
- Emits `Recall_Alert` to `Field Service` and clinical staff if an asset in the field is affected.
- Receives `Consignment_Used` event to trigger a "Pay-on-Use" Purchase Order in `Procurement`.
- Queries `Maintenance Service` for clinical equipment sterilization and calibration history.
- Emits `Inventory_Adjustment` to `Financials` for expired clinical goods.

## Key Logic

- **Case Cart Management:** Picking and preparing 100% of required items for a specific patient surgery.
- **Recall Remediation:** Instantly identifying every patient and clinical site that received a recalled lot number.
- **PAR Replenishment:** Real-time stock monitoring in clinical areas (point-of-use) to prevent stockouts of life-saving items.
- **Consignment Management:** Automating the transition from vendor-owned to hospital-owned inventory upon clinical use.
- **Cold Chain Tracking:** Ensuring clinical items (e.g., vaccines) never exceeded temperature limits during transit or storage.
- **Clinically Integrated Supply:** Mapping supply costs directly to patient cases or DRGs (Diagnosis Related Groups) for profitability analysis.
