# Project-Driven Supply Chain Service Specification

## Overview

The Project-Driven Supply Chain service coordinates supply chain activities (Procurement, Inventory, Manufacturing) within the context of specific projects. It ensures that materials are purchased, stored, and consumed against project budgets and task-level constraints. This aligns with Oracle Fusion Cloud Project-Driven Supply Chain.

## Data Model (SQLite)

- `project_inventory_reservations`: id (UUID), project_id, task_id, item_id, quantity_reserved, expiration_date.
- `project_specific_purchasing`: id (UUID), po_line_id, project_id, task_id, expenditure_type, commitment_amount.
- `project_manufacturing_links`: id (UUID), work_order_id, project_id, task_id, item_id, scheduled_date.
- `project_cost_collection_SCM`: id (UUID), source_type (PO/RECEIPT/ISSUE), source_id, project_id, task_id, amount, status (PENDING/INTERFACED).
- `project_inventory_balances`: id (UUID), project_id, task_id, item_id, quantity_on_hand, location_id.

## API Endpoints (Axum/gRPC)

- `POST /project-supply-chain/reserve`: Dedicate specific inventory to a project and task.
- `POST /project-supply-chain/purchase-link`: Associate a Purchase Order line with project financial attributes.
- `POST /project-supply-chain/issue-to-project`: Record the consumption of material against a project task.
- `GET /project-supply-chain/shortages/{project_id}`: Identify material gaps for a specific project's timeline.
- `POST /project-supply-chain/interface-costs`: Push SCM-related project costs to the Project Management financials engine.

## Inter-service Interaction

- Queries `Project Management` for project task structures, budgets, and valid expenditure types.
- Emits `Project_Commitment_Created` to `Project Control` for budgetary tracking.
- Receives `Shipment_Confirmed` from `Logistics` to trigger project-based revenue recognition.
- Queries `Inventory Service` for project-segregated vs. common stock levels.
- Emits `Cost_Incurred` event to `Project Management` (Financials).

## Key Logic

- **Inventory Segregation:** Allowing the same physical warehouse to store items that are "Owned" by different projects or "Common" stock.
- **Project-Aware Planning:** Ensuring that MRP (Supply Chain Planning) considers project due dates and specific supply sources.
- **Automated Cost Collection:** Capturing project attributes at every step (PO -> Receipt -> Issue) to ensure accurate financial reporting.
- **Commitment Tracking:** Real-time visibility into "Funds Committed" (Open POs) vs. "Actuals Incurred" for a project.
- **Inter-Project Transfers:** Managing the logic for borrowing material from one project to another and the corresponding financial cross-charges.
- **Task-Level Planning:** Constraining supply chain actions to specific project tasks (e.g., "Only buy material for Task 1.1").
