# Workforce Compensation Service Specification

## Overview

The Workforce Compensation service manages complex compensation cycles, including budget allocation, salary increases, and bonus awards based on performance and market data. This aligns with Oracle Fusion Cloud Workforce Compensation.

## Data Model (SQLite)

- `comp_plans`: id (UUID), name, start_date, end_date, budget_amount, currency, status (DRAFT/ACTIVE/COMPLETED).
- `comp_budgets`: id (UUID), comp_plan_id, department_id, allocated_amount, spent_amount.
- `comp_worksheets`: id (UUID), comp_plan_id, manager_id, status (NOT_STARTED/IN_PROGRESS/SUBMITTED/APPROVED).
- `comp_awards`: id (UUID), worksheet_id, employee_id, award_type (SALARY_INCREASE/BONUS/EQUITY), amount, percentage, effective_date, comments.
- `comp_benchmarks`: id (UUID), position_id, job_grade, market_percentile, range_min, range_max.
- `comp_approvals`: id (UUID), plan_id, approver_id, level, status, date.

## API Endpoints (Axum/gRPC)

- `POST /comp/plans`: Create a new annual or merit-based compensation plan.
- `GET /comp/worksheets/{manager_id}`: Retrieve the current compensation worksheet for a manager.
- `POST /comp/awards`: Submit individual salary/bonus recommendations.
- `GET /comp/benchmarks/{pos_id}`: Compare employee salary against market data.
- `POST /comp/approvals/submit`: Request final approval for a compensation worksheet.
- `GET /comp/reports/budget-utilization`: Track spend against allocated department budgets.

## Inter-service Interaction

- Receives `Performance_Review_Completed` from `HCM` to inform merit-based increases.
- Emits `Salary_Updated` and `Bonus_Awarded` events to `HCM/Payroll` for final processing.
- Queries `EPM Service` for company-wide budget availability.
- Queries `Identity Service` for the manager hierarchy to route approvals.

## Key Logic

- **Merit Matrices:** Automatically suggesting award percentages based on performance rating and position in the salary range (compa-ratio).
- **Budget Pooling:** Allowing managers to reallocate funds between their direct reports while staying within the total budget.
- **Off-Cycle Increases:** Handling compensation changes outside of the main annual cycle (e.g., promotions, retention).
- **Equity Administration:** Managing stock options, RSUs, and other long-term incentives as part of total compensation.
- **Pay Equity Analysis:** Identifying and flagging potential pay gaps based on gender, ethnicity, or other protected characteristics.
