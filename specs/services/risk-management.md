# Risk Management Service Specification

## Overview

The Risk Management service provides internal controls, audit monitoring, and compliance tracking. This aligns with Oracle Fusion Cloud Risk Management and Compliance.

## Data Model (SQLite)

- `risks`: id (UUID), name, category (FINANCIAL/OPERATIONAL/COMPLIANCE), status, severity (LOW/MEDIUM/HIGH).
- `controls`: id (UUID), risk_id, description, type (PREVENTIVE/DETECTIVE), owner_id.
- `incidents`: id (UUID), control_id, date, status (OPEN/REMEDIATED), description.
- `audit_logs`: id (UUID), service_name, user_id, action, timestamp, metadata (JSON).
- `compliance_frameworks`: id (UUID), name (SOX/GDPR/PCI-DSS), status.

## API Endpoints (Axum/gRPC)

- `GET /risk/incidents`: List all identified control breaches.
- `POST /risk/audit-events`: Ingest audit events from other services.
- `GET /risk/compliance-reports`: Generate compliance status for a framework.
- `POST /risk/remediation-plans`: Create tasks to address identified risks.

## Inter-service Interaction

- Receives streaming audit events from all services (via a central message bus or direct gRPC).
- Queries `Identity Service` to perform Separation of Duties (SoD) analysis on user roles.
- Emits `Control_Violation_Alert` to security and management.

## Key Logic

- **Separation of Duties (SoD):** Detecting when a single user has conflicting permissions (e.g., ability to both create and approve a payment).
- **Anomaly Detection:** Using basic statistical patterns to flag unusual financial transactions or access patterns.
- **Workflow Auditing:** Ensuring that required approval steps were actually followed for sensitive operations.
- **Regulatory Reporting:** Mapping system activities to specific regulatory requirements (e.g., SOX controls).
