# Workforce Health & Safety Service Specification

## Overview

The Workforce Health & Safety service manages the reporting, tracking, and investigation of health and safety incidents and near misses. This aligns with Oracle Fusion Cloud Workforce Health and Safety.

## Data Model (SQLite)

- `safety_incidents`: id (UUID), type (INJURY/ILLNESS/NEAR_MISS/ENVIRONMENTAL), description, date, location, reporter_id, status (NEW/INVESTIGATION/CLOSED).
- `incident_people`: id (UUID), incident_id, employee_id, role (INJURED/WITNESS/INVESTIGATOR), injury_type (optional).
- `safety_investigations`: id (UUID), incident_id, investigator_id, findings, root_cause_category, start_date, end_date.
- `corrective_actions`: id (UUID), investigation_id, description, owner_id, due_date, status (PENDING/COMPLETED).
- `safety_inspections`: id (UUID), location_id, inspector_id, date, score, findings (JSON), status.
- `safety_certifications`: id (UUID), employee_id, certification_name (e.g., OSHA, First Aid), expiry_date, status.

## API Endpoints (Axum/gRPC)

- `POST /safety/incidents`: Report a new safety incident or near miss.
- `GET /safety/incidents/{id}`: View incident details and progress.
- `POST /safety/investigations`: Record findings of a safety investigation.
- `POST /safety/corrective-actions`: Create tasks to address safety risks.
- `GET /safety/dashboard/metrics`: Retrieve key performance indicators (KPIs) for safety (e.g., TRIF, DART).
- `POST /safety/inspections`: Record results of a site safety audit.

## Inter-service Interaction

- Receives `Incident_Reported` event to notify management and safety officers.
- Queries `HCM Service` for employee details and reporting lines.
- Emits `Absence_Required` to `HCM` if an incident results in time away from work.
- Queries `Sustainability Service` for environmental impact context if relevant.
- Emits `Action_Overdue` alert to department managers.

## Key Logic

- **OSHA Reporting:** Automatically identifying incidents that meet regulatory reporting criteria (e.g., OSHA 300/301).
- **Root Cause Analysis (RCA):** Forcing structured data entry to identify if a failure was human, mechanical, or systemic.
- **Incident Escalation:** Automatically notifying senior leadership or legal for "severe" incidents.
- **Near Miss Tracking:** Encouraging reporting of potential hazards before an actual injury occurs.
- **Exposure Tracking:** Monitoring employee exposure to hazardous materials or environments over time.
- **Compliance Deadlines:** Ensuring all regulatory forms are filed within the required timeframes.
