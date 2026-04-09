# Communications Billing & Revenue (BRM) Service Specification

## Overview

The Communications Billing & Revenue Management (BRM) service provides high-scale, converged charging and billing for telecommunications, digital services, and multi-play bundles. This aligns with Oracle Communications BRM.

## Data Model (SQLite)

- `comm_billing_accounts`: id (UUID), customer_id (link to CDM), bill_cycle_day, currency, payment_method_id, status.
- `charge_offers_active`: id (UUID), account_id, offer_id (link to Launch Exp), start_date, end_date, override_price.
- `unbilled_usage_events`: id (UUID), account_id, type (DATA/VOICE/SMS/CONTENT), quantity, unit, rating_status (PENDING/RATED).
- `billing_invoices_comm`: id (UUID), account_id, period_id, amount_due, tax_amount, status (DRAFT/FINAL/PAID).
- `rating_rules_comm`: id (UUID), type, unit_price, effective_date, currency, threshold_rules (JSON).
- `converged_balances`: id (UUID), account_id, balance_type (CASH/MINUTES/DATA), current_amount, status.

## API Endpoints (Axum/gRPC)

- `POST /comm-billing/accounts`: Initialize a new subscriber billing account.
- `POST /comm-billing/usage/ingest`: Ingest high-volume usage events from the network core.
- `POST /comm-billing/rate-event`: Apply price and rating rules to a raw usage event.
- `POST /comm-billing/cycle/run`: Execute the monthly billing cycle for a group of accounts.
- `GET /comm-billing/balance/real-time`: Retrieve the current "Converged Balance" (e.g., remaining data and cash balance).
- `POST /comm-billing/adjust`: Apply a credit or debit adjustment to a subscriber's account.

## Inter-service Interaction

- Receives `Offer_Activated` from `Communications Launch Experience` to begin recurring charges.
- Emits `Bill_Finalized` event to `Financials` (AR) or Banking ERP.
- Queries `Subscription Management` for complex contract terms and multi-year renewals.
- Emits `Data_Limit_Reached` event to `Marketing` or `Digital Assistant` for upsell opportunities.
- Queries `Tax Service` for real-time tax calculation on complex bundles.

## Key Logic

- **Converged Charging:** Simultaneously managing prepaid (real-time balance) and postpaid (monthly cycle) billing on a single platform.
- **High-Scale Rating:** Processing millions of usage events per hour with sub-second rating latency.
- **Balance Management:** Maintaining distinct buckets for different resources (e.g., "Social Media Data" vs "General Data").
- **Cycle-Forward Billing:** Automatically handling customers who start their billing cycle on different days of the month.
- **Hierarchy & Sharing:** Supporting complex family plans or corporate accounts where multiple lines share a single data pool or bill.
- **Taxation Complexity:** Applying specific telecom taxes and surcharges that vary by jurisdiction and service type.
