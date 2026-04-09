# Strategic Project Portfolio Management (Project Strategic Layer) Service Specification

## Overview

The Strategic Project Portfolio Management (SPPM) service enables organizations to prioritize, select, and govern projects based on strategic alignment, financial constraints, and resource capacity. This aligns with Oracle Fusion Cloud Project Portfolio Management (Strategic Layer) and Oracle Primavera Cloud.

## Data Model (SQLite)

- `project_portfolios_strategic`: id (UUID), name, status (ACTIVE/ARCHIVED), total_funding_limit, owner_id.
- `portfolio_projects`: id (UUID), portfolio_id, name, status (PROPOSED/APPROVED/ACTIVE/COMPLETED), strategic_score (1-100), estimated_cost, estimated_revenue.
- `strategic_alignment_scores`: id (UUID), project_id, goal_id (link to EPM), score, comments.
- `portfolio_resource_capacity`: id (UUID), portfolio_id, resource_role_id, total_available_fte, period_id.
- `project_selection_scenarios`: id (UUID), portfolio_id, name, projects_included (JSON), total_npv, total_risk_score.
- `portfolio_health_metrics`: id (UUID), portfolio_id, period_id, variance_at_completion (VAC), alignment_index.

## API Endpoints (Axum/gRPC)

- `POST /project-strategy/portfolios`: Create a new strategic project portfolio.
- `POST /project-strategy/projects/propose`: Submit a project for portfolio consideration with high-level estimates.
- `POST /project-strategy/analyze/efficient-frontier`: Run an optimization engine to find the best mix of projects for a given budget.
- `GET /project-strategy/scenarios/compare`: Retrieve a side-by-side comparison of different project selection options.
- `POST /project-strategy/projects/approve`: Finalize the selection and trigger formal Project Management initialization.
- `GET /project-strategy/analytics/strategic-fit`: View how well the active project mix aligns with corporate goals (EPM).

## Inter-service Interaction

- Queries `EPM Service` (Planning) for top-down strategic goals and financial funding limits.
- Emits `Project_Approved_For_Execution` event to `Project Management` to start detailed planning.
- Queries `Resource Management` for high-level FTE availability by role.
- Receives `Project_Performance_Signal` from `Project Control` to monitor strategic health.
- Emits `Portfolio_Variance_Alert` if aggregated costs exceed the strategic budget.

## Key Logic

- **Strategic Scoring Matrix:** Automatically calculating a project's "Fit Score" based on user-defined weights for goals (e.g., "Revenue Growth" = 40%, "Risk Reduction" = 20%).
- **Efficient Frontier Optimization:** Using mathematical modeling to suggest the project set that maximizes NPV while staying within budget and resource constraints.
- **Capacity Planning (FTE Level):** Identifying if the organization has enough labor capacity (not just budget) to execute the chosen portfolio.
- **Scenario What-Ifs:** Modeling the impact of adding a "Must-Do" project or reducing the total portfolio budget by 10%.
- **Project Prioritization Ranking:** Forced-ranking of projects to identify which should be paused if funds become limited.
- **Dependency Awareness:** Tracking relationships between projects (e.g., "Project B cannot start unless Project A is in the portfolio").
