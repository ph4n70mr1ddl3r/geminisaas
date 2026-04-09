# Clinical Research (Clinical One) Service Specification

## Overview

The Clinical Research service provides a unified platform for managing clinical trials, including study design, patient enrollment, and data collection. This aligns with Oracle Clinical One (Oracle Life Sciences).

## Data Model (SQLite)

- `clinical_studies`: id (UUID), name, phase (I/II/III/IV), status (DESIGN/ACTIVE/CLOSED), sponsor_id, protocol_id.
- `study_sites`: id (UUID), study_id, facility_name, location, status, principal_investigator_id.
- `study_subjects`: id (UUID), site_id, status (SCREENING/ENROLLED/COMPLETED/DROPPED), randomization_id.
- `clinical_data_collection`: id (UUID), subject_id, form_name, fields_json (JSON), source_type (EDC/EPRO/WEARABLE), status.
- `clinical_randomization_rules`: id (UUID), study_id, algorithm_type (BLOCKED/DYNAMIC), stratification_factors (JSON).
- `clinical_supply_kits`: id (UUID), study_id, kit_type, serial_number, current_location, expiry_date, status.

## API Endpoints (Axum/gRPC)

- `POST /clinical/studies`: Define a new clinical trial protocol and design.
- `POST /clinical/subjects/enroll`: Register a new patient subject into a study site.
- `POST /clinical/data/record`: Ingest clinical trial data from an Electronic Data Capture (EDC) form.
- `GET /clinical/randomization/assign`: Retrieve the next randomized treatment assignment for a subject.
- `GET /clinical/supplies/tracking`: Monitor the inventory and movement of clinical trial kits.
- `GET /clinical/analytics/safety`: View real-time adverse event reporting across study sites.

## Inter-service Interaction

- Queries `Healthcare SCM` for clinical kit logistics and storage requirements.
- Emits `Study_Milestone_Reached` to `Financials` for sponsor billing.
- Receives `Patient_Data_Snapshot` from `Clinical Digital Assistant` if relevant to a study.
- Queries `Identity Service` for researcher credentials and IRB (Institutional Review Board) approvals.
- Emits `Adverse_Event_Detected` to global safety databases.

## Key Logic

- **Unified Data Collection:** Consolidating data from electronic forms (EDC), patient-reported outcomes (ePRO), and wearables into a single record.
- **Dynamic Randomization:** Automatically assigning subjects to treatment groups based on real-time stratification to prevent bias.
- **Blinding & Unblinding:** Ensuring that researchers and subjects remain "blinded" to treatment assignments until the study is unmasked.
- **Clinical Supply Chain:** Synchronizing drug/kit availability with patient enrollment to prevent study delays.
- **Audit Lineage (21 CFR Part 11):** Maintaining a permanent, immutable record of every change to clinical data for regulatory compliance.
- **Electronic Signatures:** Managing formal investigator sign-offs on clinical forms.
