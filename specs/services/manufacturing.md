# Manufacturing Service Specification

## Overview

The Manufacturing service manages production processes, including discrete, process, and project-based manufacturing. It integrates with Inventory for materials and Financials for costing.

## Data Model (SQLite)

- `work_orders`: id (UUID), type (DISCRETE/PROCESS), status (UNRELEASED/RELEASED/COMPLETED/CLOSED), item_id (link to Inventory), quantity, start_date, completion_date.
- `bills_of_materials` (BOM): id (UUID), assembly_item_id, component_item_id, quantity_per_assembly.
- `routings`: id (UUID), item_id, operation_sequence, work_center_id, description, standard_time.
- `work_centers`: id (UUID), name, capacity, hourly_rate (DECIMAL), resource_type (MACHINE/HUMAN).
- `production_batches`: id (UUID), work_order_id, lot_number, status, output_quantity.
- `manufacturing_costs`: id (UUID), work_order_id, resource_cost, material_cost, overhead_cost.

## API Endpoints (Axum/gRPC)

- `POST /manufacturing/work-orders`: Create and manage production work orders.
- `GET /manufacturing/bom`: Retrieve assembly/component relationships.
- `POST /manufacturing/routing`: Define production steps and work centers.
- `POST /manufacturing/complete-op`: Record completion of a production operation.
- `GET /manufacturing/cost-summary`: View actual vs. standard costs for a work order.

## Inter-service Interaction

- Queries `Inventory Service` for material availability and issues components to work orders.
- Emits `Production_Complete` event to `Inventory` to receive finished goods.
- Emits `Work_Order_Closed` event to `Financials` for final costing and variance analysis.
- Queries `Maintenance Service` for machine availability before scheduling work orders.

## Key Logic

- **Backflushing:** Automatically deduct component inventory upon assembly completion.
- **Costing Methods:** Support Standard, Actual, and FIFO costing for manufactured items.
- **Capacity Planning:** Check work center availability during work order scheduling.
- **Quality Inspections:** Integration points for checking items during the production process.
