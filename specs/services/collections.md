# Collections Service Specification

## Overview

The Collections service manages the collection of delinquent accounts, implements collection strategies, and handles customer communications. This aligns with Oracle Fusion Cloud Advanced Collections.

## Data Model (SQLite)

- `collection_cases`: id (UUID), customer_id, total_delinquent_amount, collector_id, status (NEW/ACTIVE/RESOLVED), priority.
- `collection_strategies`: id (UUID), name, description (e.g., 'Aggressive', 'Standard').
- `strategy_tasks`: id (UUID), strategy_id, step_number, action_type (CALL/LETTER/EMAIL/HOLD), days_past_due.
- `dunning_letters`: id (UUID), case_id, template_id, sent_date, recipient_email.
- `payment_promises`: id (UUID), case_id, promised_amount, promised_date, status (PENDING/KEPT/BROKEN).
- `write_offs`: id (UUID), case_id, amount, date, approval_status, reason_code.

## API Endpoints (Axum/gRPC)

- `GET /collections/delinquent-customers`: List customers with past-due invoices.
- `POST /collections/cases/assign`: Assign a delinquent case to a specific collector.
- `POST /collections/promises`: Record a promise to pay from a customer.
- `POST /collections/tasks/complete`: Mark a collection task (e.g., phone call) as completed.
- `GET /collections/reports/aging`: Generate a detailed aging report (30/60/90+ days).

## Inter-service Interaction

- Receives `Invoice_Past_Due` events from `Financials` (AR) to automatically open collection cases.
- Emits `Credit_Hold_Status` to `CRM` and `Order Management` to prevent new orders for severely delinquent customers.
- Emits `Promise_Broken_Alert` when a promised payment date passes without receipt.
- Emits `Write_Off_Posted` to `Financials` to adjust AR and record bad debt expense.

## Key Logic

- **Strategy Scoring:** Automatically assigning a collection strategy based on customer risk score and delinquency amount.
- **Workflow Automation:** Automatically sending dunning letters at predefined intervals.
- **Promise Tracking:** Real-time monitoring of payment receipts against active promises to pay.
- **Dispute Management:** Allowing collectors to flag specific invoices as 'Disputed', which pauses collection activities for that amount.
