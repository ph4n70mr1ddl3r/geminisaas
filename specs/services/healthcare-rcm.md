# Healthcare Revenue Cycle Management Service Specification

## Overview

The Healthcare Revenue Cycle Management (RCM) service manages the financial process of patient care, from registration and insurance verification to billing and collections. This aligns with Oracle Health Revenue Cycle.

## Data Model (SQLite)

- `patient_accounts_rcm`: id (UUID), patient_id (External), guarantor_id, current_balance, status (ACTIVE/CLOSED/COLLECTIONS).
- `clinical_encounters_billing`: id (UUID), patient_id, facility_id, date, primary_diagnosis_code, primary_procedure_code.
- `medical_claims`: id (UUID), encounter_id, payer_id (Insurance), total_charge, status (PENDING/DENIED/PAID), claim_number.
- `claim_denials`: id (UUID), claim_id, denial_reason_code, appeal_status (OPEN/WON/LOST), date.
- `insurance_verifications`: id (UUID), patient_id, payer_id, plan_name, coverage_status (VALID/INVALID), last_checked.
- `patient_payment_plans`: id (UUID), account_id, total_amount, monthly_installment, status.

## API Endpoints (Axum/gRPC)

- `POST /healthcare-rcm/encounters/bill`: Initiate the billing process for a completed clinical encounter.
- `POST /healthcare-rcm/claims/submit`: Generate and transmit a medical claim to a payer.
- `GET /healthcare-rcm/claims/status`: Retrieve the current adjudication status of a submitted claim.
- `POST /healthcare-rcm/verify-coverage`: Run an automated check against a payer's API for insurance eligibility.
- `GET /healthcare-rcm/analytics/denial-rate`: Identify trends in claim denials by payer or clinical area.
- `POST /healthcare-rcm/patient-billing/invoice`: Generate a statement for the patient's responsibility portion.

## Inter-service Interaction

- Receives `Encounter_Completed` event from `Clinical Digital Assistant` to trigger billing.
- Emits `Payment_Received` event to `Financials / Banking ERP` for reconciliation.
- Queries `Healthcare SCM` for cost of implants or specialized drugs used in the encounter.
- Emits `Account_Delinquent` to `Collections` for unpaid patient balances.
- Queries `Identity Service` for clinician and billing clerk permissions.

## Key Logic

- **Charge Capture Automation:** Automatically identifying billable items and services from clinical notes using AI.
- **Claim Scrubbing:** Running thousands of validation rules (e.g., NCCI edits) to ensure claims are clean before submission.
- **Denial Management:** AI-driven suggestions for appealing specific types of claim denials based on historical success rates.
- **Payer Contract Modeling:** Calculating expected reimbursement based on negotiated rates with insurance companies.
- **Propensity to Pay:** Using ML to identify patients who may need financial assistance or payment plans early in the cycle.
- **Compliance Coding:** Mapping clinical findings to the latest ICD-10 and CPT coding standards.
