# Project Billing & Revenue Service Specification

## Overview

The Project Billing & Revenue service manages customer billing, revenue recognition, and contract compliance for project-based work. This aligns with Oracle Fusion Cloud Project Billing, part of the PPM suite.

## Data Model (SQLite)

- `project_billing_contracts`: id (UUID), name, number, status (DRAFT/ACTIVE/CLOSED), customer_id, total_value, currency, start_date, end_date.
- `contract_billing_lines`: id (UUID), contract_id, billing_type (FIXED_PRICE/TIME_AND_MATERIALS/MILESTONE), amount, rate_schedule_id, status.
- `billing_events`: id (UUID), contract_id, task_id (optional), name, amount, date, status (UNBILLED/INVOICED).
- `project_invoices`: id (UUID), contract_id, invoice_number, date, amount, tax, total, status (DRAFT/SENT/PAID).
- `revenue_plans`: id (UUID), contract_id, method (AS_INVOICED/AS_INCURRED/PERCENT_COMPLETE), recognized_revenue_ytd.
- `project_rate_schedules`: id (UUID), name, role_id, hourly_rate, currency, effective_date.

## API Endpoints (Axum/gRPC)

- `POST /project-billing/contracts`: Create a new billing contract for a project.
- `POST /project-billing/events`: Record a billable event (e.g., milestone reached).
- `POST /project-billing/generate-invoice`: Aggregate billable transactions and events into a draft invoice.
- `GET /project-billing/invoices/{id}`: View detailed invoice and status.
- `POST /project-billing/recognize-revenue`: Execute revenue recognition logic based on contract plans.
- `GET /project-billing/unbilled-receivables`: Report on work performed but not yet invoiced.

## Inter-service Interaction

- Receives `Project_Cost_Recorded` from `Project Management` to trigger Time & Material billing.
- Emits `Project_Invoice_Generated` event to `Financials` (AR) for ledger recording.
- Queries `Enterprise Contracts` for legal terms and master agreements.
- Emits `Revenue_Recognized` to `Revenue Management` and `Financials`.
- Queries `Resource Management` for hourly rates and billable hours.

## Key Logic

- **Multi-Method Billing:** Supporting Fixed-Price, Time & Materials, and Milestone-based billing within the same contract.
- **Revenue vs. Billing Asynchrony:** Recognizing revenue (e.g., by effort) at a different rate than customer billing (e.g., by milestone).
- **Proration & Markup:** Automatically applying markups to labor and expense costs for customer billing.
- **Intercompany Billing:** Generating cross-charge invoices when one business unit performs work for another's project.
- **Hold & Release:** Allowing project managers to place specific billable items on hold for review before invoicing.
- **Tax Calculation:** Requesting real-time project-specific tax from `Tax Service`.
