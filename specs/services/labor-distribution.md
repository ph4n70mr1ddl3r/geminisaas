# Labor Distribution Service Specification

## Overview

The Labor Distribution service manages the complex process of distributing employee payroll costs and fringe benefits across multiple funding sources, such as grants, projects, and cost centers. This is particularly critical for Higher Education, Research, and Public Sector organizations. This aligns with Oracle Fusion Cloud Labor Distribution (part of Grants Management/PPM).

## Data Model (SQLite)

- `labor_schedules`: id (UUID), employee_id, start_date, end_date, status (ACTIVE/INACTIVE).
- `distribution_rules`: id (UUID), schedule_id, source_type (PROJECT/GRANT/GL), source_id, percentage, task_id (optional).
- `payroll_cost_input`: id (UUID), employee_id, pay_period_id, element_type (SALARY/BONUS/FRINGE), amount, currency.
- `distributed_costs_ledger`: id (UUID), input_id, distribution_rule_id, distributed_amount, gl_account_id, status (PENDING/INTERFACED).
- `effort_certification_reports`: id (UUID), employee_id, period_id, total_distributed_effort_pct, certified_by, status (PENDING/CERTIFIED).
- `distribution_adjustments`: id (UUID), original_distribution_id, adjusted_amount, reason_code, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /labor-distribution/schedules`: Define how an employee's labor costs should be split across funding sources.
- `POST /labor-distribution/calculate`: Execute the distribution engine for a payroll run.
- `GET /labor-distribution/reports/effort`: Retrieve effort certification data for a principal investigator (PI) or researcher.
- `POST /labor-distribution/adjust`: Initiate a retroactive labor distribution adjustment.
- `GET /labor-distribution/balances/{grant_id}`: View total labor costs charged against a specific grant or project.
- `POST /labor-distribution/certify`: Record a formal certification of effort for a period.

## Inter-service Interaction

- Receives `Payroll_Finalized` from `Global Payroll` to trigger the distribution process.
- Queries `Grants Management` for valid award funding periods and constraints.
- Emits `Project_Cost_Interfaced` to `Project Management` (Financials).
- Queries `HCM Service` for employee assignment and supervisor data.
- Emits `Certification_Overdue_Alert` to the research administration office.

## Key Logic

- **Percentage-Based Splitting:** Automatically dividing a single pay element (e.g., $5000 salary) across 5 different grants based on pre-defined percentages.
- **Retroactive Adjustments:** Handling "Cost Transfers" where a researcher's funding source changes in the past, requiring a reversal and re-post of payroll costs.
- **Fringe Benefit Encumbrance:** Automatically calculating and reserving funds for employer-paid benefits (Health, Pension) alongside base salary.
- **Effort Certification Workflow:** Forcing a formal review and sign-off by employees or PIs to confirm that the labor charges match the actual work performed (required for US Federal Grants).
- **Clearing Account Logic:** Temporarily holding costs in a "suspense" account if a distribution rule is missing or a project has expired.
- **Cap Enforcement:** Enforcing maximum salary caps allowed by specific sponsors (e.g., NIH salary cap).
