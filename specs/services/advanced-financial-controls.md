# Advanced Financial Controls (Risk) Service Specification

## Overview

The Advanced Financial Controls (AFC) service provides continuous monitoring of financial transactions to detect fraud, errors, and policy violations. This aligns with Oracle Fusion Cloud Advanced Financial Controls, part of the Risk Management Cloud.

## Data Model (SQLite)

- `afc_controls`: id (UUID), name, type (TRANSACTION/MASTER_DATA), status (ACTIVE/ARCHIVED), description, severity (LOW/MED/HIGH).
- `afc_filters`: id (UUID), control_id, field_name, operator (EQUALS/GREATER_THAN/IN), threshold_value.
- `afc_incidents`: id (UUID), control_id, source_transaction_id, status (PENDING/CLEARED/REMEDIATED), risk_score, first_detected, last_detected.
- `afc_remediations`: id (UUID), incident_id, type (VOID_TRANSACTION/ADJUSTMENT/EXCLUSION), owner_id, status, timestamp.
- `afc_scans`: id (UUID), start_time, end_time, total_records_scanned, incidents_found.

## API Endpoints (Axum/gRPC)

- `POST /afc/controls`: Define a new financial control with specific filters and thresholds.
- `POST /afc/scan`: Initiate a transaction scan across AP, AR, or GL for a specific period.
- `GET /afc/incidents`: Retrieve identified financial anomalies and violations.
- `POST /afc/remediate`: Record an action taken to resolve a transaction incident.
- `GET /afc/dashboard`: View financial risk trends and top control failures.

## Inter-service Interaction

- Queries `Financials Service` (AP/AR/GL) for real-time transaction data.
- Emits `Financial_Anomaly_Detected` event to the audit and finance teams.
- Queries `Identity Service` for transaction owner details.
- Emits `Control_Failure` alert to `Risk Management`.
- Queries `Procurement Service` for vendor master data anomalies (e.g., duplicate vendors).

## Key Logic

- **Continuous Monitoring:** Automatically scanning 100% of transactions rather than relying on periodic sampling.
- **Duplicate Payment Detection:** Identifying invoices with similar amounts, dates, or vendor names to prevent double payment.
- **Split Purchase Detection:** Flagging multiple small purchases that appear to be bypassing approval thresholds.
- **Ghost Employee Detection:** Matching payroll data against core HR and external records to find invalid employees.
- **Pattern Analysis:** Using AI to identify unusual spending patterns that deviate from historical norms.
- **Remediation Workflow:** Ensuring that every identified anomaly is investigated and resolved with a full audit trail.
