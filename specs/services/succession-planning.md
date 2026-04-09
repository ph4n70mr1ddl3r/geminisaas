# Succession Planning Service Specification

## Overview

The Succession Planning service enables organizations to identify and develop talent for future leadership and critical roles. This aligns with Oracle Fusion Cloud Succession Planning.

## Data Model (SQLite)

- `succession_plans`: id (UUID), role_id (link to Positions), status (ACTIVE/ARCHIVED), manager_id, plan_type (INCUMBENT/JOB).
- `succession_candidates`: id (UUID), plan_id, employee_id, readiness (READY_NOW/1-2_YEARS/3-5_YEARS), ranking, risk_of_loss (LOW/MED/HIGH), impact_of_loss.
- `talent_pools`: id (UUID), name, description, status, owner_id.
- `talent_pool_members`: id (UUID), pool_id, employee_id, date_added.
- `succession_development_tasks`: id (UUID), candidate_id, description, status (PENDING/COMPLETED), target_date.
- `nine_box_placement`: id (UUID), employee_id, potential_score (1-3), performance_score (1-3), period_id.

## API Endpoints (Axum/gRPC)

- `POST /succession/plans`: Create a new succession plan for a critical role.
- `POST /succession/candidates/add`: Add an employee as a potential successor for a plan.
- `GET /succession/plans/{id}`: View the current bench strength and candidate pipeline for a role.
- `POST /succession/talent-pools`: Create and manage groups of high-potential employees.
- `GET /succession/nine-box`: Retrieve data for the 9-Box talent matrix visualization.
- `POST /succession/readiness/update`: Update a candidate's readiness level and ranking.

## Inter-service Interaction

- Queries `HCM Service` for employee core data, performance ratings, and job history.
- Emits `Development_Goal_Suggested` event to `Goal & Performance Management`.
- Queries `Learning Service` for skills and certifications acquired by candidates.
- Emits `Candidate_Promoted` notification to executives.
- Queries `Strategic Workforce Planning` for long-term critical role forecasts.

## Key Logic

- **Bench Strength Analysis:** Quantifying the number of "Ready Now" candidates for key roles across the organization.
- **Talent Review Integration:** Feeding data into the annual Talent Review process for executive calibration.
- **Risk & Impact Modeling:** Identifying roles where the incumbent is at high risk of leaving and the impact would be severe.
- **Development Gaps:** Automatically identifying the skills a candidate needs to move from "1-2 Years" to "Ready Now."
- **Internal Mobility Support:** Prioritizing succession candidates for internal job postings.
- **Nine-Box Grid:** Visualizing talent distribution based on performance vs. potential.
