# Time and Labor Service Specification

## Overview

The Time and Labor service provides a robust environment for employee time entry, validation against corporate and legislative rules, and seamless integration with payroll. This aligns with Oracle Fusion Cloud Time and Labor.

## Data Model (SQLite)

- `time_cards`: id (UUID), employee_id, period_id, status (DRAFT/SUBMITTED/APPROVED/REJECTED), total_hours, submission_date.
- `time_entries`: id (UUID), time_card_id, date, start_time, end_time, hours, activity_type_id, project_id (link to Project Mgmt), work_order_id (link to Manufacturing).
- `time_processing_rules`: id (UUID), name, type (OVERTIME/SHIFT_PREMIUM/BREAK), logic (JSON), effective_date.
- `time_validation_rules`: id (UUID), name, error_type (WARNING/BLOCKER), logic (JSON - e.g., "Max 12 hours per day").
- `time_balances`: id (UUID), employee_id, balance_type (REGULAR/OT/HOLIDAY), total_hours, period_id.
- `time_card_approvals`: id (UUID), time_card_id, approver_id, status, comments, date.

## API Endpoints (Axum/gRPC)

- `POST /time-labor/time-cards`: Create or update a time card for a specific period.
- `GET /time-labor/time-cards/my`: Retrieve the current user's active and historical time cards.
- `POST /time-labor/submit`: Finalize a time card and trigger validation and processing rules.
- `POST /time-labor/approve`: Record manager approval for a submitted time card.
- `GET /time-labor/reports/exceptions`: View time entries that violated validation rules (e.g., missed breaks).
- `POST /time-labor/re-process`: Re-run processing rules if a policy changed retroactively.

## Inter-service Interaction

- Receives `Employee_Schedule` from `Workforce Scheduling` to pre-populate time cards.
- Receives `Absence_Approved` from `Absence Management` to automatically fill leave hours.
- Emits `Processed_Time_Data` to `Global Payroll` for earnings calculation.
- Queries `Project Management` and `Manufacturing` for valid task and work order codes.
- Emits `Labor_Costs_Incurred` to `Financials` and `Project Management` for cost allocation.

## Key Logic

- **Rule Engine:** Automatically calculating overtime (e.g., time-and-a-half after 40 hours) and shift differentials based on complex rules.
- **Redwood Time Entry:** Providing a modernized, card-based interface for rapid time entry on mobile or desktop.
- **Attestation:** Requiring employees to "sign" a digital statement during submission (e.g., "I took all required breaks").
- **Grace Periods & Rounding:** Automatically rounding clock-in/out times to the nearest 15-minute increment based on policy.
- **Project Costing:** Splitting a single day's hours across multiple projects and tasks for precise financial reporting.
- **Exception Management:** Proactively notifying managers when an employee works unplanned overtime or fails to submit their card.
