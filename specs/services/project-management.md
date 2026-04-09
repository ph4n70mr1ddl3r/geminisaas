# Project Management Service Specification

## Overview

The Project Management service (PPM) handles project creation, resource allocation, project costing, and project billing. This aligns with Oracle Fusion Cloud Project Portfolio Management.

## Data Model (SQLite)

- `projects`: id (UUID), name, description, start_date, end_date, status (ACTIVE/CLOSED/TEMPLATE), manager_id.
- `project_tasks`: id (UUID), project_id, parent_task_id, name, start_date, end_date, budget_amount.
- `project_resources`: id (UUID), project_id, employee_id, role, allocation_percentage.
- `project_costs`: id (UUID), project_id, task_id, type (LABOR/MATERIAL/EXPENSE), amount, date, source_id (link to AP or Payroll).
- `project_revenue`: id (UUID), project_id, amount, date, status (UNBILLED/BILLED).
- `project_contracts`: id (UUID), project_id, customer_id, total_value, billing_terms.

## API Endpoints (Axum/gRPC)

- `POST /projects`: Create a new project or from a template.
- `GET /projects/{id}/tasks`: List/Manage project tasks.
- `POST /projects/{id}/costs`: Record a cost against a project.
- `GET /projects/{id}/financial-summary`: Get budget vs. actuals.
- `POST /projects/{id}/billing`: Generate a project-based invoice.

## Inter-service Interaction

- Receives `Payroll_Processed` events from `HCM` to record labor costs.
- Receives `Invoice_Matched` events from `Financials` to record material/expense costs.
- Emits `Project_Invoice_Generated` to `Financials` to create a Receivable invoice.
- Queries `Identity Service` for project manager and resource details.

## Key Logic

- **Cost-to-Bill Conversion:** Capability to mark up project costs and convert them into customer invoices.
- **Budget Tracking:** Automated alerts when project costs exceed task-level budgets.
- **Resource Management:** Tracking employee utilization across multiple projects.
- **Revenue Recognition:** Supporting different methods like percentage-of-completion or milestone-based.
