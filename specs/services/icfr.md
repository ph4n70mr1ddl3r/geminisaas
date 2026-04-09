# Internal Control over Financial Reporting (ICFR) Service Specification

## Overview

The ICFR service (also known as Audit Manager) provides a framework for managing internal controls, control testing, and reporting to ensure compliance with financial regulations like SOX. This aligns with Oracle Fusion Cloud Risk Management (ICFR).

## Data Model (SQLite)

- `control_frameworks`: id (UUID), name (SOX/IFRS/GDPR), status (ACTIVE/ARCHIVED), version.
- `icfr_controls`: id (UUID), framework_id, name, description, control_type (PREVENTIVE/DETECTIVE), frequency (DAILY/WEEKLY/MONTHLY/ANNUAL), owner_id.
- `control_tests`: id (UUID), control_id, period_id, status (PENDING/IN_PROGRESS/COMPLETED/FAILED), result (PASSED/FAILED/MARGINAL), tester_id.
- `control_issues`: id (UUID), test_id, description, severity (MINOR/MAJOR/MATERIAL_WEAKNESS), status (OPEN/REMEDIATED/CLOSED).
- `audit_assessments`: id (UUID), framework_id, period_id, status (PLANNED/ACTIVE/COMPLETED), overall_result, date_finalized.
- `remediation_plans`: id (UUID), issue_id, description, due_date, status, owner_id.

## API Endpoints (Axum/gRPC)

- `POST /icfr/controls`: Define a new internal control and associate it with a framework.
- `POST /icfr/tests/schedule`: Schedule control testing for a specific period.
- `POST /icfr/tests/record`: Record the results and evidence for a control test.
- `GET /icfr/issues`: Retrieve all open control issues and deficiencies.
- `POST /icfr/remediate`: Submit a remediation plan for an identified issue.
- `GET /icfr/compliance-report`: Generate a summary of control effectiveness for management or external auditors.

## Inter-service Interaction

- Queries `Advanced Access Controls (AAC)` for SOD violation data to include in assessments.
- Receives `Control_Failure` events from other services to automatically trigger an ICFR incident.
- Queries `Identity Service` for control owners and testers.
- Emits `Material_Weakness_Alert` to the Audit Committee and CFO.
- Queries `Enterprise Data Management (EDM)` for legal entity and cost center context.

## Key Logic

- **Test Cycle Management:** Orchestrating the workflow for planning, performing, and reviewing control tests.
- **Deficiency Aggregation:** Automatically identifying "Material Weaknesses" if multiple related controls fail.
- **Evidence Management:** Providing a secure repository for attaching audit evidence (e.g., screenshots, logs, reports) to a test.
- **Regulatory Mapping:** Linking specific system controls to high-level regulatory requirements (e.g., mapping a "Journal Approval" control to SOX Section 404).
- **Remediation Workflow:** Ensuring that every control failure has a documented plan and timeline for resolution.
- **Automated Control Testing:** Using AI to continuously monitor transactions (e.g., checking for journals without approval) instead of periodic manual testing.
