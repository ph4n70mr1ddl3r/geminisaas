# HCM Service Specification

## Overview

The HCM (Human Capital Management) service manages the complete employee lifecycle, including Core HR, Payroll, Absence Management, Time and Labor, Talent Management, and Compensation. This aligns with Oracle Fusion Cloud HCM.

## Data Model (SQLite)

- `employees`: id (UUID), first_name, last_name, email, hire_date, position_id, department_id, manager_id, status (ACTIVE/LEAVE/TERMINATED).
- `departments`: id (UUID), name, location.
- `positions`: id (UUID), title, salary_range_min, salary_range_max, grade.
- `payroll_runs`: id (UUID), period_id, status (PENDING/PROCESSED), total_amount.
- `pay_slips`: id (UUID), employee_id, payroll_run_id, gross_amount, net_amount, tax_deductions, benefit_deductions.
- `absences`: id (UUID), employee_id, type (VACATION/SICK/MATERNITY), start_date, end_date, status (PENDING/APPROVED/REJECTED).
- `time_cards`: id (UUID), employee_id, period_id, total_hours, status (DRAFT/SUBMITTED/APPROVED).
- `time_entries`: id (UUID), time_card_id, project_id (optional), date, hours, activity_type.
- `performance_goals`: id (UUID), employee_id, description, target_date, status (NOT_STARTED/IN_PROGRESS/COMPLETED).
- `performance_reviews`: id (UUID), employee_id, reviewer_id, period_id, rating (1-5), comments.
- `compensation_plans`: id (UUID), employee_id, base_salary, bonus_percentage, effective_date.

## API Endpoints (Axum/gRPC)

- `GET /hcm/employees`: List/Search employees.
- `POST /hcm/employees`: Hire a new employee.
- `POST /hcm/payroll/process`: Run payroll for a specific period.
- `POST /hcm/absences/request`: Submit a leave request.
- `POST /hcm/time/submit`: Submit a weekly time card.
- `GET /hcm/talent/reviews/{employee_id}`: View performance history.
- `GET /hcm/org-chart`: Retrieve organizational structure.

## Inter-service Interaction

- Emits `Payroll_Processed` event to `Financials` to create journal entries for salary expenses.
- Queries `Identity Service` to create/revoke user accounts for employees.
- Emits `Labor_Costs_Incurred` to `Project Management` and `Manufacturing` based on approved time cards.
- Receives `Employee_Reward_Recommended` from `Project Management` for bonus processing.

## Key Logic

- **Accrual Engine:** Automatically calculating vacation and sick leave balances based on tenure and plan rules.
- **Org Hierarchy:** Ability to traverse the manager/employee relationships for approvals.
- **Retroactive Pay:** Calculating salary adjustments for past periods if a pay increase was delayed.
- **Time Validation:** Ensuring time entries do not exceed 24 hours in a day and align with project/work order allocations.
- **Rating Normalization:** Adjusting performance ratings across different departments to ensure consistent evaluation standards.
