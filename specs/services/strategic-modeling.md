# Strategic Modeling Service Specification

## Overview

The Strategic Modeling service provides long-term, multi-year financial forecasting and "what-if" scenario modeling. It enables corporate finance teams to simulate the impact of high-level strategic decisions (e.g., acquisitions, divestitures, capital structure changes). This aligns with Oracle Fusion Cloud Strategic Modeling (EPM).

## Data Model (SQLite)

- `strategic_models`: id (UUID), name, status (DRAFT/FINAL), period_horizon_years, base_currency.
- `model_scenarios`: id (UUID), model_id, name (OPTIMISTIC/PESSIMISTIC/BASELINE), assumption_set (JSON).
- `strategic_accounts`: id (UUID), model_id, account_name, account_type (REVENUE/EXPENSE/ASSET/LIABILITY), formula_logic (JSON).
- `model_input_variables`: id (UUID), scenario_id, variable_name (e.g., 'Inflation Rate'), value, period_id.
- `strategic_rollups`: id (UUID), parent_model_id, child_model_id, ownership_pct, status.
- `debt_scheduler_modeling`: id (UUID), model_id, principal, interest_rate_type (FIXED/VAR), repayment_terms (JSON).

## API Endpoints (Axum/gRPC)

- `POST /strategic-modeling/models`: Create a new long-term financial model.
- `POST /strategic-modeling/calculate`: Execute the modeling engine for all defined scenarios.
- `GET /strategic-modeling/scenarios/compare`: Retrieve a side-by-side comparison of different financial outcomes.
- `POST /strategic-modeling/rollup`: Consolidate child business unit models into a parent corporate model.
- `GET /strategic-modeling/what-if/simulate`: Run a real-time simulation based on a modified input variable (e.g., 2% higher tax).
- `POST /strategic-modeling/finalize`: Lock a model version for board-level reporting.

## Inter-service Interaction

- Queries `EPM Service` (Planning) for short-term budget and forecast data to use as a starting point.
- Emits `Strategic_Target_Updated` event to `Planning` to align operational budgets with long-term goals.
- Queries `Financials` for historical actuals to establish model baselines.
- Emits `Cash_Requirement_Forecast` to `Treasury` for long-term liquidity planning.
- Queries `Tax Reporting` for effective tax rate assumptions.

## Key Logic

- **Goal Seeking:** Automatically identifying the required revenue growth or cost reduction needed to achieve a target net income or cash balance.
- **Scenario Management:** Maintaining distinct sets of assumptions (e.g., different interest rate environments) without duplicating the core model logic.
- **Complex Debt Modeling:** Simulating the impact of new bond issuances or loan repayments on interest expense and cash flow.
- **Entity Rollups:** Handling complex ownership percentages and minority interests during consolidation.
- **Trailing Twelve Months (TTM):** Automatically calculating rolling metrics for trend analysis.
- **Natural Language Analysis:** Using the "Planning Agent" to explain why a scenario resulted in a specific financial outcome.
