# Subscription Management Service Specification

## Overview

The Subscription Management service handles recurring billing, renewals, and lifecycle management for subscription-based products and services. This aligns with Oracle Fusion Cloud Subscription Management.

## Data Model (SQLite)

- `subscriptions`: id (UUID), customer_id, product_id, status (ACTIVE/EXPIRED/CANCELLED), start_date, next_billing_date, total_contract_value.
- `subscription_plans`: id (UUID), name, frequency (MONTHLY/ANNUALLY), base_price, trial_period_days.
- `subscription_billing_lines`: id (UUID), subscription_id, period_start, period_end, amount, status (PENDING/INVOICED).
- `subscription_amendments`: id (UUID), subscription_id, change_type (UPGRADE/DOWNGRADE/PAUSE), effective_date.
- `usage_records`: id (UUID), subscription_id, usage_type, quantity, date (for metered billing).

## API Endpoints (Axum/gRPC)

- `POST /subscriptions`: Create a new subscription for a customer.
- `POST /subscriptions/{id}/amend`: Modify an existing subscription (e.g., add seats).
- `POST /subscriptions/generate-billing`: Trigger the billing engine to create invoices for the current period.
- `POST /subscriptions/{id}/renew`: Manually or automatically renew an expiring subscription.
- `GET /subscriptions/reports/mrr`: Calculate Monthly Recurring Revenue (MRR) and Churn Rate.

## Inter-service Interaction

- Queries `Product Hub / Inventory` for subscription-enabled products.
- Emits `Subscription_Invoice_Required` to `Financials` (AR) to generate the actual invoice.
- Emits `Provisioning_Request` to external systems or internal services when a subscription is activated.
- Emits `Revenue_Contract_Impact` to `Revenue Management` when a subscription is modified.

## Key Logic

- **Proration Engine:** Automatically calculating partial-period charges when a subscription starts or changes mid-month.
- **Metered Billing:** Integrating with usage tracking systems to bill based on consumption (e.g., data used, API calls).
- **Grace Periods & Dunning:** Coordinating with the `Collections Service` for failed subscription payments.
- **Evergreen Subscriptions:** Supporting contracts that automatically renew until explicitly cancelled.
