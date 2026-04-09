# Joint Venture Management Service Specification

## Overview

The Joint Venture Management (JVM) service manages complex joint venture accounting, specifically for industries like Oil & Gas, Mining, and Construction. It handles cost distribution among partners based on ownership percentages. This aligns with Oracle Fusion Cloud Joint Venture Management.

## Data Model (SQLite)

- `joint_ventures`: id (UUID), name, status (ACTIVE/CLOSED), manager_id, start_date, end_date.
- `joint_venture_partners`: id (UUID), jv_id, partner_id (link to Customers/Vendors), ownership_percentage, billing_address.
- `jv_distributable_costs`: id (UUID), jv_id, source_journal_id, original_amount, distributed_amount, status (PENDING/DISTRIBUTED).
- `jv_distributions`: id (UUID), cost_id, partner_id, amount, billing_period_id.
- `jv_billing_notices`: id (UUID), partner_id, amount, date, status (SENT/PAID).

## API Endpoints (Axum/gRPC)

- `POST /jv`: Create a new joint venture and define ownership shares.
- `POST /jv/identify-costs`: Automatically scan GL/AP for costs that should be distributed to JV partners.
- `POST /jv/calculate-split`: Distribute identified costs across partners based on their ownership percentages.
- `POST /jv/generate-billing`: Create receivable invoices (AR) for each partner's share of costs.
- `GET /jv/reports/partner-statement`: Generate a detailed cost and billing statement for a specific partner.

## Inter-service Interaction

- Queries `Financials` (GL/AP) for distributable costs based on project codes or cost centers.
- Emits `JV_Distribution_Generated` to `Financials` (AR) to create invoices for partners.
- Queries `Project Management` for project-level costs associated with a venture.
- Receives `Partner_Payment_Received` from `Financials` to update venture funding status.

## Key Logic

- **Ownership Splitting:** Automatically dividing a single expense into multiple partner-level transactions.
- **Minimum Thresholds:** Rules to prevent billing for very small amounts (rolling them over to the next period).
- **Overhead Calculation:** Automatically adding a management fee or overhead percentage to distributed costs.
- **Intercompany Handling:** Specialized logic for when a partner is also an internal legal entity.
