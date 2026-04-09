# Strategic Workforce Planning Service Specification

## Overview

The Strategic Workforce Planning service enables organizations to analyze their current workforce, forecast future demand for skills, and develop long-term talent strategies. This aligns with Oracle Fusion Cloud Strategic Workforce Planning.

## Data Model (SQLite)

- `workforce_plans`: id (UUID), name, status (DRAFT/APPROVED), planning_horizon_years (1-10), total_budget.
- `skills_inventory`: id (UUID), employee_id (link to HCM), skill_name, proficiency_level (1-5), last_verified.
- `demand_forecasts`: id (UUID), plan_id, department_id, period_id, headcount_required, skills_required (JSON).
- `supply_forecasts`: id (UUID), plan_id, department_id, period_id, headcount_projected (including attrition), skills_projected.
- `gap_analysis`: id (UUID), plan_id, period_id, headcount_gap, skills_gap (JSON).
- `workforce_initiatives`: id (UUID), gap_id, type (HIRE/RESKILL/TRANSFER/RETIRE), description, target_impact.

## API Endpoints (Axum/gRPC)

- `POST /workforce-planning/plans`: Create a new strategic workforce plan.
- `GET /workforce-planning/skills-assessment`: View aggregated skill levels across the organization.
- `POST /workforce-planning/run-forecast`: Execute demand and supply projections.
- `GET /workforce-planning/gaps/{plan_id}`: Retrieve results of a gap analysis.
- `POST /workforce-planning/initiatives`: Create action plans to close workforce gaps.
- `GET /workforce-planning/what-if/automation`: Simulate how AI agents will replace or augment human roles.

## Inter-service Interaction

- Queries `HCM Service` for real-time headcount, demographics, and skills profiles.
- Queries `EPM Service` (Planning) for financial budgets to constrain workforce plans.
- Emits `Initiative_Created` event to `Recruiting` and `Learning` for execution.
- Receives `Attrition_Event` from `HCM` to update supply forecasts.
- Queries `Project Management` for future project pipelines to inform resource demand.

## Key Logic

- **Skill Gap Analysis:** Identifying the delta between current workforce capabilities and the skills needed to meet future business goals.
- **Attrition Modeling:** Using historical data and AI to predict future employee turnover by department or role.
- **Digital Worker Augmentation:** Planning for the "Hybrid Workforce" where human roles are augmented by AI agents.
- **What-If Scenarios:** Simulating the impact of mergers, acquisitions, or entering new markets on workforce requirements.
- **Recruiting vs. Reskilling:** Comparing the cost and time impact of hiring new talent versus training existing employees.
