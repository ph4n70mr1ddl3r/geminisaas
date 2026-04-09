# Incentive Compensation Service Specification

## Overview

The Incentive Compensation service manages the calculation and distribution of sales commissions, bonuses, and awards based on performance against quotas and targets. This aligns with Oracle Fusion Cloud Incentive Compensation.

## Data Model (SQLite)

- `ic_plans`: id (UUID), name, status (ACTIVE/DRAFT), start_date, end_date, currency.
- `ic_plan_components`: id (UUID), plan_id, name, component_type (COMMISSION/BONUS), weight_pct, formula_id.
- `ic_quotas`: id (UUID), participant_id, component_id, target_amount, period_id.
- `ic_credit_rules`: id (UUID), name, criteria (JSON), credit_receiver_pct, status.
- `ic_earnings`: id (UUID), participant_id, source_order_id (link to Order Mgmt), component_id, amount, status (CALCULATED/APPROVED/PAID).
- `ic_participants`: id (UUID), employee_id (link to HCM), role (SALES_REP/MANAGER/EXEC), start_date, status.
- `ic_attainment`: id (UUID), participant_id, period_id, quota_pct, components_summary (JSON).

## API Endpoints (Axum/gRPC)

- `POST /incentive-compensation/plans`: Define a new incentive plan with multiple components and formulas.
- `POST /incentive-compensation/calculate`: Run the calculation engine for a specific period and group of participants.
- `GET /incentive-compensation/earnings/my`: Retrieve current earned commissions for the logged-in rep.
- `POST /incentive-compensation/dispute`: Initiate a dispute for a specific earning or credit.
- `GET /incentive-compensation/attainment-dashboard`: View quota attainment trends across the sales team.
- `POST /incentive-compensation/approve-payments`: Finalize earnings for payroll processing.

## Inter-service Interaction

- Receives `Order_Fulfilled` from `Order Management` to trigger commission crediting.
- Emits `Commission_Finalized` event to `Global Payroll` for payment.
- Queries `CRM Service` for opportunity and account metadata used in credit rules.
- Queries `Sales Performance Management` for territory and hierarchy context.
- Emits `Performance_Alert` to managers for under-performing reps.

## Key Logic

- **Credit Allocation:** Using complex rules to determine who gets credit for a sale (e.g., split credits between territories).
- **Formula Engine:** Executing tiered commission rates (e.g., 5% up to 100% quota, 10% for over-attainment).
- **Draws & Recoveries:** Managing non-recoverable and recoverable draws against future earnings.
- **Clawbacks:** Automatically reversing earnings if an order is returned or a payment is not received.
- **Quota Roll-up:** Aggregating individual attainment into team and regional performance views.
- **What-If Estimator:** Providing reps with a calculator to estimate their commission for a pending deal.
