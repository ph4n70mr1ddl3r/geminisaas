# Sales Performance Management Service Specification

## Overview

The Sales Performance Management (SPM) service handles territory management, quota planning, and incentive compensation. This aligns with Oracle Fusion Cloud Sales Performance Management.

## Data Model (SQLite)

- `sales_territories`: id (UUID), name, region_code, manager_id, parent_territory_id.
- `territory_accounts`: id (UUID), territory_id, customer_id (link to CRM).
- `sales_quotas`: id (UUID), sales_rep_id, period_id, amount, currency, status (DRAFT/APPROVED).
- `compensation_plans`: id (UUID), name, start_date, end_date, formula_id.
- `incentive_commissions`: id (UUID), sales_rep_id, order_id (link to CRM), amount, status (PENDING/CALCULATED/PAID).
- `attainment_metrics`: id (UUID), sales_rep_id, period_id, quota_attainment_pct, bonus_earned.

## API Endpoints (Axum/gRPC)

- `GET /spm/territories`: View and manage sales territory structure.
- `POST /spm/quotas`: Assign quotas to sales representatives.
- `GET /spm/commissions/{rep_id}`: View historical and projected commissions.
- `POST /spm/calculate-incentives`: Trigger the commission calculation engine.
- `GET /spm/leaderboard`: Retrieve sales performance rankings.

## Inter-service Interaction

- Receives `Order_Fulfilled` from `CRM` to trigger commission calculation.
- Emits `Commission_Paid` event to `HCM/Payroll` for bonus processing.
- Queries `CRM Service` for historical sales data to assist in quota planning.
- Receives `Employee_Hired/Terminated` from `HCM` to manage sales rep status.

## Key Logic

- **Territory Alignment:** Logic to ensure every customer account is assigned to exactly one (or more) territories without gaps or overlaps.
- **Incentive Engine:** Calculating complex commissions based on product type, deal size, and quota attainment (e.g., accelerators).
- **Quota Roll-up:** Automatically aggregating individual quotas into team and regional quotas.
- **Draws and Caps:** Managing advanced compensation rules such as recoverable/non-recoverable draws and commission caps.
- **Credit Assignment:** Determining which sales reps get credit for a deal when multiple parties are involved (split credits).
