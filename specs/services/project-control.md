# Project Control Service Specification

## Overview

The Project Control service provides advanced budgeting and forecasting for project-based work, ensuring financial targets are met throughout the project lifecycle. This aligns with Oracle Fusion Cloud Project Control.

## Data Model (SQLite)

- `project_budgets`: id (UUID), project_id (link to Project Mgmt), budget_type (COST/REVENUE), name, status (DRAFT/BASELINED), version, total_amount.
- `project_forecasts`: id (UUID), project_id, period_id, forecast_type (EAC/ETC), amount, date, status.
- `budget_lines`: id (UUID), budget_id, task_id, resource_id, period_id, amount, quantity.
- `forecast_lines`: id (UUID), forecast_id, task_id, resource_id, period_id, amount, quantity.
- `financial_performance`: id (UUID), project_id, period_id, actual_cost, budgeted_cost, variance, performance_index (CPI/SPI).
- `budget_baselines`: id (UUID), budget_id, baseline_date, baselined_by, version_notes.

## API Endpoints (Axum/gRPC)

- `POST /project-control/budgets`: Create a new budget for a project.
- `POST /project-control/budgets/baseline`: Set a budget as the "gold standard" for performance measurement.
- `POST /project-control/forecasts/generate`: Automatically generate a project forecast based on actuals and budget.
- `GET /project-control/performance/{project_id}`: Retrieve real-time budget vs. actuals analysis.
- `POST /project-control/forecasts/update`: Manually adjust a project forecast for a specific period.
- `GET /project-control/kpis`: View key performance indicators (e.g., Earned Value) for a portfolio of projects.

## Inter-service Interaction

- Receives `Project_Cost_Recorded` from `Project Management` to update actuals in performance reports.
- Emits `Budget_Exceeded` alert to project managers and `Procurement`.
- Queries `EPM Service` for top-down corporate budget constraints.
- Emits `Forecast_Finalized` event to `Financials` for corporate cash-flow planning.
- Queries `Resource Management` for future labor costs based on committed assignments.

## Key Logic

- **Earned Value Management (EVM):** Calculating metrics like Cost Performance Index (CPI) and Schedule Performance Index (SPI) to predict final project outcomes.
- **Budget Baselining:** Locking a specific version of a budget to track variances over time as the project progresses.
- **ETC/EAC Calculation:** Automatically calculating "Estimate to Complete" (ETC) and "Estimate at Completion" (EAC) based on current performance trends.
- **Change Management:** Tracking how budget revisions impact the overall project margin and delivery date.
- **Time-Phased Planning:** Spreading budget and forecast amounts across periods (e.g., monthly) to manage cash flow.
