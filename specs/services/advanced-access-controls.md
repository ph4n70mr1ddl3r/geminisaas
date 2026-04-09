# Advanced Access Controls (Risk) Service Specification

## Overview

The Advanced Access Controls (AAC) service provides "Zero Trust" security monitoring for the ERP environment, specifically managing Segregation of Duties (SOD) and sensitive access risk. This aligns with Oracle Fusion Cloud Advanced Access Controls.

## Data Model (SQLite)

- `risk_rules`: id (UUID), name, type (SOD/SENSITIVE_ACCESS), status (ACTIVE/ARCHIVED), description, severity (LOW/MED/HIGH).
- `risk_rule_conditions`: id (UUID), rule_id, entitlement_1_id, entitlement_2_id (only for SOD), logic (AND/OR).
- `entitlements`: id (UUID), name, permission_ids (JSON), role_ids (JSON).
- `access_incidents`: id (UUID), rule_id, user_id, status (PENDING/CLEARED/REMEDIATED), risk_score, first_detected, last_detected.
- `access_remediations`: id (UUID), incident_id, type (REVOKE_ROLE/EXCLUSION/APPROVAL), owner_id, status, timestamp.
- `access_requests`: id (UUID), user_id, requested_role_id, status (PENDING/APPROVED/DENIED), justification, sod_risk_score.

## API Endpoints (Axum/gRPC)

- `POST /aac/rules`: Define a new SOD or sensitive access rule.
- `POST /aac/scan`: Run a comprehensive access risk scan across all users.
- `GET /aac/incidents`: Retrieve a list of identified access violations.
- `POST /aac/remediate`: Record an action taken to resolve an access incident.
- `POST /aac/simulate`: Predict the risk of an access request before it is approved.
- `GET /aac/dashboard`: View overall risk levels and outstanding incident counts.

## Inter-service Interaction

- Queries `Identity Service` for user roles, permissions, and hierarchical relationships.
- Emits `Access_Violation_Detected` event to the security and audit teams.
- Receives `User_Provisioning_Requested` from `Identity Service` for pre-approval SOD analysis.
- Queries `Financial Consolidation (FCCS)` to monitor access in reporting environments.
- Emits `Control_Failure` alert to `Risk Management`.

## Key Logic

- **Segregation of Duties (SOD):** Identifying cases where a single user has conflicting permissions (e.g., "Enter Invoice" and "Approve Payment").
- **Sensitive Access Monitoring:** Tracking users who have high-risk permissions (e.g., "Create General Ledger Period").
- **Real-Time Simulation:** Providing a "What-If" analysis for new access requests to prevent risks before they occur.
- **Remediation Tracking:** Ensuring that every identified risk is either resolved, accepted (with justification), or mitigated.
- **Audit Trails for Access:** Maintaining an immutable history of who had what access and when, for regulatory compliance.
- **Role Hierarchies:** Recursively checking inherited permissions to find hidden SOD risks.
