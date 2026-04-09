# Pharmacovigilance (Argus Safety) Service Specification

## Overview

The Pharmacovigilance service manages the intake, processing, and regulatory reporting of adverse events related to medical products. This aligns with Oracle Argus Safety (Oracle Life Sciences).

## Data Model (SQLite)

- `safety_cases`: id (UUID), case_number, reporter_id, source (PATIENT/PHYSICIAN/STUDY), status (NEW/TRIAGE/DATA_ENTRY/MEDICAL_REVIEW/CLOSED), receipt_date.
- `adverse_events`: id (UUID), case_id, event_description, meddra_code, severity (SERIOUS/NON_SERIOUS), outcome (RECOVERED/FATAL).
- `case_products`: id (UUID), case_id, product_id, suspect_flag (BOOL), dosage, administration_route.
- `safety_reports`: id (UUID), case_id, type (E2B/CIOMS/FDA), status (GENERATED/SUBMITTED), destination_authority (FDA/EMA/PMDA).
- `medical_assessments`: id (UUID), case_id, causality_score, expectedness_status, seriousness_criteria (JSON).
- `safety_workflow_tasks`: id (UUID), case_id, task_type (TRIAGE/QUALITY_CHECK), owner_id, status, due_date.

## API Endpoints (Axum/gRPC)

- `POST /safety/cases`: Record a new adverse event case from any source.
- `POST /safety/cases/triage`: Perform initial prioritization and seriousness check.
- `POST /safety/meddra/code`: Auto-suggest MedDRA (Medical Dictionary for Regulatory Activities) codes for events.
- `POST /safety/reports/generate`: Create a regulatory submission file (e.g., E2B R3) for a specific case.
- `GET /safety/analytics/signals`: Identify potential safety signals using statistical disproportionality.
- `POST /safety/cases/medical-review`: Record a medical doctor's assessment of causality and risk.

## Inter-service Interaction

- Receives `Adverse_Event_Detected` from `Clinical Research` to automatically open a safety case.
- Queries `Healthcare SCM` for product lot numbers and manufacturer data.
- Emits `Regulatory_Submission_Completed` event for audit tracking.
- Queries `Identity Service` for medical reviewer credentials.
- Emits `Safety_Signal_Alert` to product managers and regulatory affairs.

## Key Logic

- **Case Processing Workflow:** Forcing a strict, multi-step lifecycle for every safety report to ensure no data is missed.
- **Auto-Coding (MedDRA):** Using AI to map natural language descriptions of symptoms to the standardized MedDRA medical terminology.
- **Seriousness Determination:** Automatically flagging cases that meet regulatory criteria for "Serious" events (e.g., hospitalization, death).
- **Electronic Submissions (E2B):** Generating and transmitting XML files to global health authorities via secure gateways.
- **Causality Assessment:** Providing structured frameworks for determining if a suspect product caused an adverse event.
- **Global Compliance Calendar:** Automatically calculating submission deadlines based on the receipt date and event seriousness (e.g., "7-day report").
