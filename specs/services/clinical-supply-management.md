# Clinical Trial Supply Management (CTSM) Service Specification

## Overview

The Clinical Trial Supply Management (CTSM) service manages the specialized logistics, inventory, and dispensing of clinical trial materials. It ensures that the right kits are available at the right site for the right patient, while maintaining strict blinding and regulatory compliance. This aligns with Oracle Fusion Cloud CTSM (Life Sciences).

## Data Model (SQLite)

- `clinical_kits_master`: id (UUID), kit_type, treatment_type (DRUG/PLACEBO/DEVICE), storage_conditions (JSON), expiration_date.
- `kit_inventory_site`: id (UUID), kit_id, site_id (link to Clinical Research), status (AVAILABLE/DISPENSED/QUARANTINED), lot_id.
- `kit_dispensation_logs`: id (UUID), kit_id, subject_id, visit_id, date_dispensed, dispensed_by_id.
- `supply_replenishment_plans`: id (UUID), site_id, kit_type, target_inventory_level, reorder_point, current_qty.
- `kit_shipment_requests`: id (UUID), origin_dc_id, target_site_id, kit_list_json (JSON), tracking_number, status.
- `temperature_logs_kits`: id (UUID), shipment_id, reading_date, temperature_c, threshold_violated (BOOL).

## API Endpoints (Axum/gRPC)

- `POST /clinical-supply/inventory/receive`: Record the arrival of clinical kits at a study site.
- `POST /clinical-supply/dispense`: Record the allocation of a kit to a patient subject while maintaining blinding.
- `GET /clinical-supply/replenishment/calculate`: Run the replenishment engine to identify sites with low inventory.
- `POST /clinical-supply/quarantine`: Place a lot or specific kits on hold (e.g., due to temperature excursion).
- `GET /clinical-supply/site-health`: Retrieve inventory and expiration metrics for a clinical site.
- `POST /clinical-supply/return`: Record the return or destruction of unused/expired clinical materials.

## Inter-service Interaction

- Receives `Subject_Enrolled` from `Clinical Research` to initialize site-level supply demand.
- Emits `Kit_Dispensed` event to `Clinical Research` to update the patient encounter record.
- Queries `Logistics` for real-time tracking of kit shipments to remote sites.
- Emits `Temperature_Excursion_Alert` to clinical monitors and safety teams.
- Queries `Healthcare SCM` for specialized warehousing and manufacturer data.

## Key Logic

- **Blinded Replenishment:** Automatically ordering kits without revealing whether the site is low on Active Drug or Placebo.
- **Randomization Linkage:** Integrating with the IRT (Interactive Response Technology) to ensure the dispensed kit matches the patient's treatment arm.
- **Excursion Management:** Automatically identifying and quarantining kits that were exposed to temperatures outside of the validated range.
- **Lot Expiration Tracking:** Proactively suggesting shipments to clear sites of kits nearing their "Do Not Dispense" date.
- **Pool Management:** Sharing a single supply pool across multiple related clinical trials to reduce waste.
- **Compliance Documentation:** Maintaining a "Chain of Custody" for every individual kit from manufacturing to final patient dispensation.
