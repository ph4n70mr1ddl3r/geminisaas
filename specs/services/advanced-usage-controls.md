# Advanced Usage Controls (Risk) Service Specification

## Overview

The Advanced Usage Controls (AUC) service provides continuous monitoring of user activity and system configuration changes to ensure data privacy, security, and operational integrity. This aligns with Oracle Fusion Cloud Advanced Usage Controls, part of the Risk Management Cloud.

## Data Model (SQLite)

- `usage_controls`: id (UUID), name, status (ACTIVE/ARCHIVED), type (USER_ACTIVITY/SYSTEM_CONFIG/API_USAGE), severity (LOW/MED/HIGH).
- `activity_logs`: id (UUID), user_id, action_type (LOGIN/DATA_EXPORT/PERMISSION_CHANGE), timestamp, metadata_json.
- `usage_incidents`: id (UUID), control_id, log_id, status (PENDING/CLEARED/REMEDIATED), risk_score, timestamp.
- `data_lineage_trace`: id (UUID), record_id, source_service, target_service, action, user_id, timestamp.
- `config_snapshots`: id (UUID), service_name, config_key, config_value, changed_by, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /auc/controls`: Define a new usage control to monitor for suspicious patterns (e.g., mass data downloads).
- `GET /auc/incidents`: Retrieve identified user activity violations or risky configuration changes.
- `POST /auc/remediate`: Record an action taken to resolve a usage incident.
- `GET /auc/data-lineage/{id}`: Retrieve the full history of who accessed or modified a specific data record across services.
- `GET /auc/dashboard`: View overall system usage risk and top risky users.

## Inter-service Interaction

- Queries **ALL Services** for real-time audit logs and configuration snapshots.
- Emits `Suspicious_Activity_Detected` event to the security and legal teams.
- Queries `Identity Service` for user role and session metadata.
- Emits `Control_Failure` alert to `Risk Management`.
- Receives `API_Call_Event` from the API gateway to monitor for brute-force or injection attempts.

## Key Logic

- **Continuous Audit Readiness:** Automatically capturing and storing immutable records of all "sensitive" actions across the ERP.
- **Mass Export Detection:** Identifying and blocking cases where a user attempts to download an unusually large volume of customer or financial data.
- **Configuration Drift Monitoring:** Alerting when a critical system setting (e.g., a tax rule or a bank account) is changed without a corresponding ticket.
- **Data Provenance:** Tracking the "Golden Record" movement to ensure data privacy (GDPR/CCPA) compliance.
- **AI-Driven Baseline:** Establishing a "normal" usage profile for each user role and flagging deviations.
- **Audit Visualization:** Providing graphical depictions of access paths and data flows for internal and external auditors.
