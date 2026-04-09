# Process Manufacturing Service Specification

## Overview

The Process Manufacturing service manages production for industries where goods are produced in batches using recipes and formulas (e.g., Chemicals, Food & Beverage, Pharmaceuticals). It handles ingredient scaling, co-products, by-products, and rigorous quality/regulatory compliance. This aligns with Oracle Fusion Cloud Process Manufacturing.

## Data Model (SQLite)

- `batch_work_orders`: id (UUID), recipe_id, status (PENDING/RELEASED/COMPLETED/CLOSED), planned_qty, actual_qty, start_date, end_date.
- `recipes_process`: id (UUID), name, version, item_id (finished good), status (ACTIVE/OBSOLETE), scale_type (LINEAR/FIXED).
- `formulas_process`: id (UUID), recipe_id, ingredients_json (JSON - items and quantities), co_products_json (JSON), by_products_json (JSON).
- `routings_process`: id (UUID), recipe_id, operation_sequence, work_center_id, duration_min, temperature_req, pressure_req.
- `batch_steps`: id (UUID), batch_id, step_number, status (NOT_STARTED/IN_PROGRESS/COMPLETED), actual_start, actual_end.
- `material_potency`: id (UUID), item_id, lot_number, potency_pct, expiry_date.

## API Endpoints (Axum/gRPC)

- `POST /process-mfg/batches`: Create a new production batch from a recipe.
- `POST /process-mfg/batches/{id}/scale`: Automatically adjust ingredient quantities based on the target batch size or a primary ingredient's potency.
- `GET /process-mfg/recipes`: Retrieve master recipes and formula details.
- `POST /process-mfg/steps/complete`: Record completion of a specific batch step, including actual resource and material usage.
- `GET /process-mfg/yield-analysis`: Compare planned vs actual yields for co-products and by-products.
- `POST /process-mfg/yield/record`: Record the output of finished goods, co-products, and by-products.

## Inter-service Interaction

- Queries `Inventory Service` for raw material lots and issues ingredients based on potency.
- Emits `Batch_Completed` event to `Inventory` to receive multiple output items.
- Emits `Quality_Check_Required` to `Quality Management` for in-process samples.
- Queries `Maintenance Service` for sterilized or calibrated equipment availability.
- Emits `Consumption_Data` to `Cost Accounting` for batch-level variance analysis.

## Key Logic

- **Formula Scaling:** Automatically adjusting all ingredient quantities when the batch size changes, while respecting fixed vs. proportional scaling rules.
- **Potency-Based Adjustments:** Increasing or decreasing an ingredient's quantity based on its active ingredient concentration (e.g., if a chemical is 90% pure instead of 95%).
- **Co-Product & By-Product Management:** Tracking multiple outputs from a single production run (e.g., cream and skim milk from whole milk).
- **Yield Calculation:** Predicting and measuring the loss of material during complex chemical or physical transformations.
- **Batch Traceability:** Maintaining a rigorous link between raw material lots and finished product batches for regulatory compliance (FDA/EMA).
- **Environmental Constraints:** Enforcing temperature, pressure, or humidity ranges during specific batch steps.
