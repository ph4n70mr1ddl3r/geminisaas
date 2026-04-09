# Benefits Management Service Specification

## Overview

The Benefits Management service manages employee benefit plans, eligibility rules, and annual/life-event enrollment cycles. This aligns with Oracle Fusion Cloud Benefits.

## Data Model (SQLite)

- `benefit_plans`: id (UUID), name, type (HEALTH/RETIREMENT/LIFE/VOLUNTARY), status (ACTIVE/INACTIVE), start_date, end_date.
- `benefit_options`: id (UUID), plan_id, name (e.g., PPO/HMO), coverage_details (JSON), employer_cost, employee_cost.
- `benefit_eligibility_rules`: id (UUID), plan_id, criteria (JSON - e.g., age, tenure, role, location).
- `benefit_enrollments`: id (UUID), employee_id, option_id, status (ENROLLED/WAIVED/PENDING), start_date, end_date.
- `benefit_life_events`: id (UUID), employee_id, type (MARRIAGE/BIRTH/TRANSFER), status (UNPROCESSED/PROCESSED/MANUAL_REVIEW), event_date.
- `benefit_beneficiaries`: id (UUID), enrollment_id, name, relationship, percentage.

## API Endpoints (Axum/gRPC)

- `GET /benefits/eligible-plans/{employee_id}`: Retrieve a list of benefit plans available to a specific employee.
- `POST /benefits/enroll`: Register an employee's selection for a benefit option.
- `GET /benefits/my-enrollments`: View active benefit coverage for the current user.
- `POST /benefits/record-life-event`: Report a life event that may trigger an enrollment window.
- `GET /benefits/plan-comparison`: Retrieve detailed comparison data for multiple benefit options.
- `POST /benefits/calculate-costs`: Run the cost engine to determine employee and employer contributions.

## Inter-service Interaction

- Queries `HCM Service` for employee core data (age, tenure, location) to determine eligibility.
- Emits `Benefit_Deduction_Updated` event to `Global Payroll` for payroll processing.
- Receives `Employee_Hired/Terminated` from `HCM/Recruiting` to trigger enrollment windows or coverage ends.
- Queries `Identity Service` for relationship details (dependents).
- Emits `Enrollee_Data` to external benefit providers (e.g., insurance carriers).

## Key Logic

- **Eligibility Engine:** Automatically evaluating complex rules to determine which employees can see and select specific plans.
- **Life Event Processing:** Identifying when a change in HR data (e.g., address change to a different state) should trigger a new enrollment window.
- **Open Enrollment Orchestration:** Managing mass enrollment windows, including automated reminders and status tracking.
- **Evidence of Insurability (EOI):** Managing the workflow for plans that require medical approval before coverage is granted.
- **Flexible Credits:** Handling "Flex Credit" models where employees are given a budget to spend across various benefits.
- **COBRA & Portability:** Automating the transition of benefits for terminated employees.
