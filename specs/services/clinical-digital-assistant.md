# Clinical Digital Assistant Service Specification

## Overview

The Clinical Digital Assistant (CDA) service provides a secure, voice-enabled AI interface for healthcare providers to document patient encounters, query clinical data, and automate administrative tasks. This aligns with Oracle Clinical Digital Assistant (Oracle Health).

## Data Model (SQLite)

- `cda_sessions`: id (UUID), provider_id, patient_id (External Link), status (RECORDING/TRANSCRIBING/COMPLETED), date.
- `encounter_transcripts`: id (UUID), session_id, raw_audio_url, text_transcript, detected_intent (SOAP_NOTE/ORDER/REFERRAL).
- `clinical_soap_notes`: id (UUID), session_id, subjective, objective, assessment, plan, status (DRAFT/SIGNED).
- `cda_task_automations`: id (UUID), session_id, task_type (SCHEDULE_FOLLOW_UP/CREATE_ORDER/BILLING_CODE), status.
- `medical_billing_suggestions`: id (UUID), soap_note_id, icd_code, cpt_code, confidence_score.
- `provider_preferences_cda`: id (UUID), provider_id, language, template_id, auto_sign_threshold.

## API Endpoints (Axum/gRPC)

- `POST /clinical-assistant/session/start`: Begin a voice-recorded clinical encounter.
- `POST /clinical-assistant/transcript/ingest`: Submit audio data for real-time transcription and clinical entity extraction.
- `GET /clinical-assistant/soap-note/{id}`: Retrieve an AI-generated SOAP note draft for provider review.
- `POST /clinical-assistant/task/confirm`: Approve an AI-suggested follow-up action (e.g., ordering a lab test).
- `GET /clinical-assistant/billing-codes`: Retrieve AI-suggested ICD-10/CPT codes based on encounter text.
- `POST /clinical-assistant/sign-note`: Electronically sign and finalize a clinical note for integration with external EHR.

## Inter-service Interaction

- Queries `Healthcare SCM` for medical equipment or supply availability during an encounter.
- Emits `Order_Requested` event to `Healthcare SCM` for surgical kits or high-value implants.
- Emits `Service_Bill_Created` event to `Financials` (AR) or Banking ERP for patient billing.
- Queries `Identity Service` for clinician credentials and signing authority.
- Receives `Patient_Schedule` from external clinical systems to prepare for daily sessions.

## Key Logic

- **Clinical Entity Extraction:** Identifying medical terms (medications, diagnoses, symptoms) within natural language transcripts.
- **SOAP Note Generation:** Automatically organizing a conversation into the standard Subjective, Objective, Assessment, and Plan format.
- **Ambient Listening:** High-accuracy voice capture designed for noisy clinical environments with multiple speakers.
- **Billing Code Suggestion:** Using NLP to match clinical findings to the most specific and accurate billing codes.
- **Voice-to-Task Automation:** Translating verbal commands (e.g., "Schedule a follow-up in two weeks") into executable system events.
- **HIPAA Compliance:** Ensuring all voice and text data is encrypted and handled according to strict healthcare privacy regulations.
