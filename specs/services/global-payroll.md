# Global Payroll Service Specification

## Overview

The Global Payroll service provides localized payroll processing, calculations, and regulatory reporting for multiple countries and jurisdictions. This aligns with Oracle Fusion Cloud Global Payroll.

## Data Model (SQLite)

- `payroll_definitions`: id (UUID), name, frequency (MONTHLY/BI_WEEKLY), start_date, status (ACTIVE/ARCHIVED).
- `payroll_runs`: id (UUID), definition_id, period_id, status (PENDING/CALCULATED/VALIDATED/PROCESSED/VOID), start_time, end_time.
- `payroll_elements`: id (UUID), employee_id, element_type (BASIC_SALARY/OVERTIME/TAX/BENEFIT), amount, currency, effective_date, source_id.
- `payroll_balances`: id (UUID), employee_id, balance_type (GROSS/NET/TAX_YTD), amount, period_id.
- `retroactive_pay_adjustments`: id (UUID), employee_id, original_period_id, adjustment_amount, reason (SALARY_INCREASE/ERROR), status (PENDING/PROCESSED).
- `payroll_costing_segments`: id (UUID), run_id, cost_center_id, gl_account_id, amount, debited_credited (BOOL).

## API Endpoints (Axum/gRPC)

- `POST /payroll/runs`: Initiate a new payroll run for a specific definition and period.
- `GET /payroll/runs/{id}/status`: Monitor the progress of a payroll calculation and validation.
- `POST /payroll/calculate`: Execute the payroll engine to compute gross-to-net for all employees in a run.
- `POST /payroll/process-payments`: Finalize a payroll run and trigger bank transfers.
- `GET /payroll/slips/{employee_id}`: Retrieve a detailed pay slip for a specific period.
- `POST /payroll/retro`: Calculate and apply retroactive pay adjustments for past periods.

## Inter-service Interaction

- Queries `HCM Service` for employee core data, positions, and base salaries.
- Receives `Workforce_Compensation_Awarded` from `Workforce Compensation` to update pay elements.
- Receives `Time_Card_Approved` from `HCM/WFM` to calculate overtime and shift differentials.
- Emits `Payroll_Processed` event to `Financials` (GL) to create salary expense and liability journal entries.
- Queries `Identity Service` for payroll administrator permissions.
- Emits `Payment_Initiated` to `Treasury` for bank file generation.

## Key Logic

- **Gross-to-Net Engine:** Calculating earnings, taxes, and deductions based on legislative and corporate rules.
- **Retroactive Pay:** Automatically calculating and applying pay adjustments for past periods if a salary change was dated in the past.
- **Localized Legislative Rules:** Applying specific tax tables, insurance rules, and mandatory deductions for each country/region.
- **Payroll Costing:** Distributing payroll expenses across multiple cost centers or projects based on employee labor allocation.
- **Validation Engine:** Automatically flagging anomalies (e.g., "Salary > 2x average") before final processing.
- **Statutory Reporting:** Generating required tax and insurance filings for government bodies (e.g., W-2, P60).
