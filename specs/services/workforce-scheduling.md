# Workforce Scheduling Service Specification

## Overview

The Workforce Scheduling service provides automated, AI-driven shift planning and optimization to ensure the right number of employees with the right skills are available at the right time. This aligns with Oracle Fusion Cloud Workforce Scheduling.

## Data Model (SQLite)

- `schedule_templates`: id (UUID), name, department_id, period_type (WEEKLY/MONTHLY), required_roles (JSON).
- `work_shifts`: id (UUID), template_id, shift_name, start_time, end_time, duration_hours, capacity_required.
- `employee_schedules`: id (UUID), employee_id, shift_id, date, status (PUBLISHED/DRAFT/CANCELLED).
- `employee_availability`: id (UUID), employee_id, day_of_week, start_time, end_time, is_available (BOOL).
- `scheduling_requests`: id (UUID), employee_id, date, type (TIME_OFF/SHIFT_SWAP), target_shift_id, status (PENDING/APPROVED/DENIED).
- `scheduling_metrics`: id (UUID), department_id, date, labor_cost, capacity_utilization, understaffed_hours.

## API Endpoints (Axum/gRPC)

- `POST /scheduling/optimize`: Run the AI scheduling engine to automatically generate shifts based on demand.
- `GET /scheduling/my-schedule`: Retrieve current and future shifts for the logged-in employee.
- `POST /scheduling/shift-swap`: Initiate a shift swap request between two employees.
- `POST /scheduling/availability`: Update an employee's recurring availability.
- `POST /scheduling/publish`: Finalize and communicate a draft schedule to the workforce.
- `GET /scheduling/demand-forecast`: View predicted labor demand for a specific period.

## Inter-service Interaction

- Queries `HCM Service` for employee core data, roles, skills, and reporting lines.
- Receives `Absence_Approved` from `HCM` to automatically update availability and shift capacity.
- Queries `Manufacturing/Service` for production plans or service demand to inform labor requirements.
- Emits `Shift_Assigned` notification to employees via `Digital Assistant` or `Mobile`.
- Queries `Financials` for labor budget constraints and actual cost analysis.

## Key Logic

- **AI Optimization:** Automatically assigning shifts to employees based on skills, availability, cost, and fairness rules.
- **Demand-Based Planning:** Using historical and forecasted demand (e.g., foot traffic, order volume) to determine required headcount per hour.
- **Constraint Handling:** Enforcing labor laws (e.g., maximum hours per week, mandatory rest periods) and corporate policies.
- **Shift Swapping Logic:** Validating that the replacement employee has the required skills before approving a swap.
- **Fairness & Seniority:** Considering employee preferences and tenure in the scheduling algorithm.
- **Real-Time Adjustments:** Re-calculating schedules in response to unexpected absences or demand spikes.
