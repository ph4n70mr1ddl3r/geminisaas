# Quality Management Service Specification

## Overview

The Quality Management service handles inspections, non-conformances, and corrective actions (CAPA) across the supply chain. This aligns with Oracle Fusion Cloud Quality Management.

## Data Model (SQLite)

- `inspection_plans`: id (UUID), item_id, process_step (RECEIVE/WORK_IN_PROGRESS/FINAL), characteristics (JSON - e.g., 'Weight between 10-11g').
- `inspection_results`: id (UUID), inspection_plan_id, inspector_id, status (PASS/FAIL), measured_values (JSON), date.
- `non_conformances`: id (UUID), source_id (Receipt/Work Order), severity, description, status (OPEN/INVESTIGATING/CLOSED).
- `corrective_actions` (CAPA): id (UUID), non_conformance_id, root_cause, action_plan, owner_id.
- `quality_test_methods`: id (UUID), name, equipment_required, standard_procedure_ref.

## API Endpoints (Axum/gRPC)

- `POST /quality/inspections`: Record results for a quality check.
- `GET /quality/inspections/pending`: List items awaiting inspection (e.g., in a quarantine area).
- `POST /quality/non-conformances`: Log a quality issue found during production or receiving.
- `POST /quality/capa`: Create a corrective action plan for a major quality failure.
- `GET /quality/reports/defect-rate`: Calculate the percentage of failed inspections by item or vendor.

## Inter-service Interaction

- Receives `Receipt_Recorded` from `Procurement` for items requiring incoming inspection.
- Receives `Production_Complete` from `Manufacturing` for final product testing.
- Emits `Item_Quarantined` to `Inventory` to prevent the use/sale of failed items.
- Emits `Vendor_Quality_Alert` to `Procurement` when a supplier consistently fails inspections.

## Key Logic

- **Sampling Plans:** Automatically determining how many units to test based on lot size (e.g., AQL - Acceptable Quality Level).
- **Skip-Lot Inspection:** Automatically reducing inspection frequency for vendors with a high historical pass rate.
- **Root Cause Analysis:** Providing structured templates (e.g., 5 Whys, Fishbone) for investigating non-conformances.
- **Dispositioning:** Rules for what to do with failed items (Rework, Scrap, Return to Vendor, Accept as-is).
