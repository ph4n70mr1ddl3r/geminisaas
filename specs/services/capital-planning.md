# Capital Planning Service Specification

## Overview

The Capital Planning service manages the evaluation, prioritization, and tracking of major capital expenditures (CapEx). It provides a structured workflow for requesting funds for new assets, projects, or infrastructure. This aligns with Oracle Fusion Cloud EPM (Capital Planning).

## Data Model (SQLite)

- `capital_requests`: id (UUID), title, description, requestor_id, department_id, status (SUBMITTED/REVIEW/APPROVED/REJECTED), total_estimated_cost, roi_pct.
- `investment_types`: id (UUID), name (NEW_ASSET/UPGRADE/REPLACEMENT/INTANGIBLE), depreciation_method_default.
- `capital_project_link`: id (UUID), request_id, project_id (link to Project Mgmt), status.
- `asset_life_assumptions`: id (UUID), category_id, useful_life_years, salvage_value_pct.
- `capital_impact_forecast`: id (UUID), request_id, period_id, depreciation_expense, maintenance_cost, incremental_revenue.
- `prioritization_scores`: id (UUID), request_id, strategic_fit (1-5), risk_score (1-5), financial_score (1-5), overall_rank.

## API Endpoints (Axum/gRPC)

- `POST /capital-planning/requests`: Submit a new capital expenditure proposal.
- `GET /capital-planning/requests/search`: Search for capital requests based on department, cost, or status.
- `POST /capital-planning/evaluate`: Record scoring and financial analysis for a pending request.
- `GET /capital-planning/portfolio-view`: Retrieve an aggregated view of all approved and pending capital projects.
- `POST /capital-planning/approve`: Transition a request to approved status and trigger project/asset creation.
- `GET /capital-planning/analytics/spend-vs-budget`: Analyze actual CapEx spend against the approved capital budget.

## Inter-service Interaction

- Queries `Financials` (Fixed Assets) for historical asset performance data to inform replacement requests.
- Emits `Capital_Project_Approved` event to `Project Management` to initiate a capital project.
- Queries `EPM Service` (Planning) for total CapEx budget availability by department.
- Emits `New_Asset_Scheduled` event to `Fixed Assets` for future capitalization.
- Queries `Maintenance Service` for asset health data to validate replacement needs.

## Key Logic

- **ROI/NPV Calculation:** Automatically calculating Net Present Value (NPV) and Internal Rate of Return (ROI) for proposed investments.
- **Lease vs. Buy Analysis:** Comparing the long-term financial impact of leasing an asset versus purchasing it outright.
- **Impact on Financial Statements:** Automatically forecasting the future depreciation and cash flow impact of an approved request.
- **Workflow Orchestration:** Multi-level approval routing based on request amount and strategic importance.
- **Asset Lifecycle Continuity:** Seamlessly transitioning a capital request into a formal Project, then into a commissioned Fixed Asset.
- **Scenario Comparison:** Evaluating multiple investment options (e.g., "Option A: Repair" vs "Option B: Replace") side-by-side.
