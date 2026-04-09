# Sales & Operations Planning (S&OP) Service Specification

## Overview

The Sales & Operations Planning (S&OP) service provides a unified platform for executive-level planning, aligning demand, supply, and financial goals. It facilitates the monthly S&OP cycle through scenario modeling and collaborative consensus. This aligns with Oracle Fusion Cloud S&OP.

## Data Model (SQLite)

- `sop_plans`: id (UUID), name, status (DRAFT/CONCENSUS/FINAL), planning_horizon_months, owner_id.
- `demand_scenarios_sop`: id (UUID), plan_id, name (UNCONSTRAINED/CONSTRAINED), qty_json (JSON), total_revenue.
- `supply_scenarios_sop`: id (UUID), plan_id, name, capacity_utilization_pct, production_qty_json (JSON).
- `sop_consensus_forecast`: id (UUID), plan_id, period_id, item_category_id, consensus_qty, status.
- `sop_variance_analysis`: id (UUID), plan_id, period_id, forecast_vs_actual_qty, bias_pct.
- `sop_meeting_artifacts`: id (UUID), plan_id, agenda_type (DEMAND_REVIEW/SUPPLY_REVIEW/EXEC), decisions_taken (JSON).

## API Endpoints (Axum/gRPC)

- `POST /sop/plans/create`: Initialize a new monthly S&OP planning cycle.
- `POST /sop/scenarios/simulate`: Run a "what-if" supply simulation for a given demand profile.
- `GET /sop/concensus/report`: Retrieve the final agreed-upon forecast across all departments.
- `POST /sop/decisions/record`: Log formal decisions and action items from the S&OP executive meeting.
- `GET /sop/analytics/gap-to-budget`: Visualize the difference between the operational plan and the financial budget (EPM).

## Inter-service Interaction

- Queries `Supply Chain Planning` for unconstrained demand forecasts and current supply constraints.
- Queries `EPM Service` (Planning) for financial revenue and margin targets.
- Emits `Consensus_Plan_Finalized` event to `Inventory / Manufacturing` to drive execution.
- Queries `Marketing` for upcoming major promotion impacts.
- Emits `Strategic_Shortage_Alert` to the SCM Command Center.

## Key Logic

- **Consensus Building:** Providing a structured workflow for Sales, Supply Chain, and Finance to arrive at a single number.
- **Rough-Cut Capacity Planning (RCCP):** Checking if the long-term plan is feasible against critical resources without detailed shop-floor scheduling.
- **Financial Alignment:** Converting the quantity-based consensus plan into monetary values (Revenue, COGS, Margin) in real-time.
- **Aggregation & Disaggregation:** Allowing planners to work at the Product Family/Region level and automatically pushing those targets down to individual items.
- **Multi-Scenario Comparison:** Comparing "Growth Strategy" vs "Cost-Cutting" scenarios side-by-side.
- **Cycle Governance:** Forcing the sequence of reviews (Product -> Demand -> Supply -> Financial -> Executive).
