# HCM Service Specification

## Overview

The HCM (Human Capital Management) service manages employee records, payroll, benefits, and performance.

## Data Model (SQLite)

- `employees`: id (UUID), first_name, last_name, email, hire_date, position_id, department_id, manager_id.
- `departments`: id (UUID), name, location.
- `positions`: id (UUID), title, salary_range_min, salary_range_max.
- `payroll_runs`: id (UUID), period_id, status (PENDING/PROCESSED), total_amount.
- `pay_slips`: id (UUID), employee_id, payroll_run_id, amount, tax_deductions.

## API Endpoints (Axum/gRPC)

- `GET /hcm/employees`: List/Search employees.
- `POST /hcm/employees`: Hire a new employee.
- `GET /hcm/payroll`: View/Process payroll.
- `GET /hcm/org-chart`: Retrieve organizational structure.

## Inter-service Interaction

- Emits `Payroll_Processed` event to `Financials` to create journal entries for salary expenses.
- Queries `Identity Service` to create user accounts for new employees.
- Emits `Employee_Terminated` to revoke access in `Identity Service`.

## Key Logic

- **Org Hierarchy:** Ability to traverse the manager/employee relationships.
- **Privacy:** Strict RBAC on employee records (salary, performance reviews).
- **Tax Calculation:** Modular tax calculation logic based on employee location (complex, requires external service or plugins).
