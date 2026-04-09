# People Leader Workbench Service Specification

## Overview

The People Leader Workbench service provides a strategic, AI-enhanced workspace for managers to lead their teams, align talent with business goals, and manage the full employee lifecycle in a single view. This aligns with the Oracle People Leader Workbench (2026 roadmap).

## Data Model (SQLite)

- `manager_dashboards`: id (UUID), manager_id, team_size, open_requisitions_count, budget_utilization_pct.
- `team_skills_matrix`: id (UUID), manager_id, skill_name, average_proficiency, gap_count.
- `talent_health_indicators`: id (UUID), employee_id, attrition_risk (LOW/MED/HIGH), engagement_score, last_feedback_date.
- `team_budget_modeling`: id (UUID), manager_id, period_id, allocated_salary_budget, planned_increases, remaining_funds.
- `leader_action_items`: id (UUID), manager_id, type (REVIEW/ONBOARD/APPROVE), priority, due_date, status.
- `team_milestone_calendar`: id (UUID), manager_id, event_type (ANNIVERSARY/BIRTHDAY/CERT_EXPIRY), date.

## API Endpoints (Axum/gRPC)

- `GET /leader-workbench/team-overview`: Retrieve a summary of team performance, skills, and health.
- `GET /leader-workbench/skills-gap`: Identify critical skills missing from the team to meet upcoming goals.
- `POST /leader-workbench/budget/simulate`: Model the impact of a team reorganization or mass merit increase.
- `POST /leader-workbench/feedback/bulk`: Send quick, personalized recognition to multiple team members using GenAI assistance.
- `GET /leader-workbench/hiring-pipeline`: View the status of all open positions and candidates for the manager's team.
- `POST /leader-workbench/retention/plan`: Create a personalized retention strategy for a high-risk, high-impact employee.

## Inter-service Interaction

- Queries `HCM Service` for core employee data, leave balances, and performance ratings.
- Queries `Workforce Compensation` for budget and salary range data.
- Emits `Hiring_Request_Initiated` to `Recruiting Service`.
- Queries `Oracle Grow` for team development paths and learning progress.
- Emits `Team_Celebration_Event` to `Oracle Celebrate`.
- Queries `Workforce Scheduling` for team availability and shift coverage.

## Key Logic

- **Strategic Talent Alignment:** Helping managers identify if they have the right people to execute their department's quarterly goals.
- **Attrition Prediction:** Using AI to flag employees who show behavioral patterns associated with leaving (e.g., decreased engagement, finished all learning).
- **Manager Coaching Agents:** Providing AI-driven suggestions on how to handle difficult conversations or reward top performers.
- **Unified Action Center:** Consolidating approvals and tasks from Payroll, Absences, Recruiting, and Performance into one list.
- **Compensation Fairness:** Highlighting potential pay inequities within the team before annual cycles begin.
- **Succession Visibility:** Allowing managers to see potential internal successors for their own or their direct reports' roles.
