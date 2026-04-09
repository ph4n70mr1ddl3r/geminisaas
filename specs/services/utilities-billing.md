# Utilities Customer Care & Billing (C2M) Service Specification

## Overview

The Utilities Customer Care & Billing service manages utility customer relationships, meter data, and complex billing for electricity, gas, water, and waste services. This aligns with Oracle Utilities Customer to Meter (C2M).

## Data Model (SQLite)

- `utility_accounts`: id (UUID), customer_id (link to CDM), service_point_id, status (ACTIVE/FINAL/PENDING), rate_schedule_id.
- `service_points`: id (UUID), location_gis (JSON), service_type (E/G/W), meter_id (link to Utilities Mgmt).
- `meter_reads_billing`: id (UUID), meter_id, read_date, read_value, read_type (ACTUAL/ESTIMATED/ADJUSTED).
- `bill_segments`: id (UUID), account_id, period_start, period_end, amount, consumption_qty, tax_amount.
- `utility_payments`: id (UUID), account_id, amount, method (ACH/CC/CASH), status, date.
- `rate_components`: id (UUID), rate_schedule_id, description, tier_start, tier_end, price_per_unit.

## API Endpoints (Axum/gRPC)

- `POST /utilities-billing/accounts`: Set up a new utility customer account and link to a service point.
- `POST /utilities-billing/meter-reads`: Ingest meter readings for billing purposes.
- `POST /utilities-billing/generate-bill`: Run the billing engine for a specific account or cycle.
- `GET /utilities-billing/account-balance`: Retrieve the real-time financial status of a utility customer.
- `POST /utilities-billing/stop-service`: Finalize an account and generate a final bill.
- `GET /utilities-billing/consumption-history`: View historical usage patterns for an account.

## Inter-service Interaction

- Receives `Telemetry_Ingested` from `Utilities Management` to use as actual meter reads.
- Emits `Bill_Finalized` event to `Financials` (AR) for general ledger recording.
- Queries `CDM Service` for master customer profile and communication preferences.
- Emits `Non_Payment_Notice` to `Collections` if a bill is past due.
- Queries `Identity Service` for customer portal access.

## Key Logic

- **Tiered Rate Engine:** Calculating bills based on complex usage tiers (e.g., first 500 kWh at $0.10, next 500 at $0.12).
- **Time-of-Use (TOU) Billing:** Applying different prices depending on when the resource was consumed (Peak vs. Off-Peak).
- **Net Metering:** Handling customers who produce energy (e.g., solar) and credit their bills accordingly.
- **Budget Billing:** Smoothing out seasonal fluctuations by allowing customers to pay a fixed monthly amount based on annual averages.
- **Meter-to-Cash Workflow:** Automating the full cycle from receiving a meter ping to confirming a payment.
- **Estimated Billing:** Automatically generating a bill based on historical averages if an actual reading is unavailable.
