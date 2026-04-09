# Collections Service Specification

## Overview

The Collections service manages the collection of delinquent accounts, implements collection strategies, and handles customer communications. This aligns with Oracle Fusion Cloud Advanced Collections, including 2026 features like AI-driven "Credit to Cash" optimization and consolidated invoicing.

## Data Model (SQLite)

- `collection_cases`: id (UUID), customer_id, total_delinquent_amount, collector_id, status (NEW/ACTIVE/RESOLVED), priority, risk_score.
- `collection_strategies`: id (UUID), name, description, ai_optimized (BOOL), target_segment.
- `strategy_tasks`: id (UUID), strategy_id, step_number, action_type (CALL/LETTER/EMAIL/HOLD/LEGAL), days_past_due.
- `dunning_letters`: id (UUID), case_id, template_id, sent_date, recipient_email.
- `payment_promises`: id (UUID), case_id, promised_amount, promised_date, status (PENDING/KEPT/BROKEN).
- `write_offs`: id (UUID), case_id, amount, date, approval_status, reason_code.
- `invoice_disputes`: id (UUID), invoice_id, dispute_reason, status (OPEN/INVESTIGATION/RESOLVED), amount_disputed, date.
- `consolidated_bill_groups`: id (UUID), customer_id, group_name, billing_frequency, aggregation_rules (JSON).

## API Endpoints (Axum/gRPC)

- `GET /collections/delinquent-customers`: List customers with past-due invoices, prioritized by the "Credit to Cash" AI advisor.
- `POST /collections/cases/assign`: Assign a delinquent case to a specific collector or an automated AI agent.
- `POST /collections/promises`: Record a promise to pay from a customer.
- `POST /collections/dispute`: Create or track a receivable dispute.
- `POST /collections/consolidate-invoices`: Group multiple invoices into a single bill based on fulfillment events.
- `GET /collections/reports/aging`: Generate a detailed aging report with 2026 predictive analytics for "likely to pay".

## Inter-service Interaction

- Receives `Invoice_Past_Due` events from `Financials` (AR) to automatically open collection cases.
- Emits `Credit_Hold_Status` to `CRM` and `Order Management` to prevent new orders.
- Emits `Promise_Broken_Alert` when a promised payment date passes.
- Emits `Write_Off_Posted` to `Financials` to adjust AR.
- Queries `CDM (Customer Data Management)` for master customer profiles and hierarchy.

## Key Logic

- **Strategy Scoring:** Automatically assigning a collection strategy based on AI risk modeling and past payment behavior.
- **Workflow Automation:** Automatically sending dunning letters and escalating to "Legal" or "Third-Party" based on strategy steps.
- **Consolidated Billing:** Aggregating invoices from different departments or projects into a single customer statement to reduce friction.
- **Dispute Resolution Workflow:** Managing the lifecycle of a dispute, including routing to sales or warehouse for investigation.
- **Predictive Churn/Bad Debt:** Analyzing trends to identify customers likely to default before they go past due.
