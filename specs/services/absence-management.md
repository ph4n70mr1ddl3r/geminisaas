# Absence Management Service Specification

## Overview

The Absence Management service manages all types of employee leave, including vacation, sick time, maternity/paternity, and jury duty. This aligns with Oracle Fusion Cloud Absence Management.

## Data Model (SQLite)

- `absence_types`: id (UUID), name, category (VACATION/SICK/UNPAID), status (ACTIVE/INACTIVE), validation_rules (JSON).
- `absence_plans`: id (UUID), name, type (ACCRUAL/QUALIFICATION/NO_ENTITLEMENT), accrual_frequency, rollover_limit.
- `absence_records`: id (UUID), employee_id, type_id, start_date, end_date, duration_hours, status (PENDING/APPROVED/REJECTED/WITHDRAWN), reason_code.
- `absence_accruals`: id (UUID), employee_id, plan_id, period_id, balance_start, earned, taken, balance_end.
- `absence_certifications`: id (UUID), record_id, doc_type (MEDICAL/LEGAL), status (REQUIRED/RECEIVED/EXPIRED), due_date.
- `absence_donations`: id (UUID), from_employee_id, to_employee_id, amount, date.

## API Endpoints (Axum/gRPC)

- `POST /absences/request`: Submit a new leave request.
- `GET /absences/balances/{employee_id}`: Retrieve current accrual balances for all active plans.
- `POST /absences/approve`: Record manager approval or rejection of a leave request.
- `GET /absences/history`: List past and future absences for an employee.
- `POST /absences/calculate-accruals`: Trigger the accrual engine for a specific period.
- `GET /absences/calendar`: Retrieve a team-level view of planned absences to check for conflicts.

## Inter-service Interaction

- Queries `HCM Service` for employee tenure, role, and location to calculate accrual rates.
- Emits `Absence_Approved` event to `Global Payroll` to adjust pay (e.g., unpaid leave).
- Emits `Absence_Calendar_Update` to `Workforce Scheduling` to prevent shift assignments.
- Queries `Identity Service` for the manager hierarchy to route approvals.
- Receives `Workforce_Health_Incident` from `WHS Service` which may trigger an automated sick leave record.

## Key Logic

- **Accrual Engine:** Automatically calculating leave earned based on tenure, hours worked, or flat rates.
- **Entitlement Calculation:** Determining if an employee qualifies for specific leave types (e.g., FMLA) based on legislative rules.
- **Scheduling Conflict Detection:** Identifying when a leave request overlaps with a mandatory work event or shift.
- **Retroactive Adjustments:** Handling cases where a leave request is edited after the payroll period has closed.
- **Absence Cascading:** Automatically deducting from "Sick" before moving to "Unpaid" based on policy.
- **Donation Pools:** Managing "Leave Banks" where employees can donate hours to a shared pool for colleagues in need.
