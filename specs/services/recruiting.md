# Recruiting & Candidate Experience Service Specification

## Overview

The Recruiting service manages the end-to-end talent acquisition process, from job requisition creation to candidate onboarding. This aligns with Oracle Fusion Cloud Recruiting.

## Data Model (SQLite)

- `job_requisitions`: id (UUID), title, description, department_id (link to HCM), hiring_manager_id, status (DRAFT/OPEN/FILLED/CANCELLED), location, target_start_date.
- `candidates`: id (UUID), first_name, last_name, email, phone, resume_url, skills (JSON), source (LINKEDIN/REFERRAL/CAREER_SITE).
- `job_applications`: id (UUID), requisition_id, candidate_id, status (APPLIED/SCREENING/INTERVIEW/OFFER/HIRED/REJECTED), application_date.
- `interviews`: id (UUID), application_id, interviewer_id, scheduled_time, duration_min, feedback_score (1-5), comments.
- `job_offers`: id (UUID), application_id, salary_amount, bonus_pct, start_date, expiration_date, status (PENDING/EXTENDED/ACCEPTED/DECLINED).
- `onboarding_checklists`: id (UUID), candidate_id, task_name, due_date, status (PENDING/COMPLETED).

## API Endpoints (Axum/gRPC)

- `POST /recruiting/requisitions`: Create a new job opening.
- `GET /recruiting/career-site/jobs`: List open positions for external candidates.
- `POST /recruiting/apply`: Submit a candidate application.
- `POST /recruiting/interviews/schedule`: Arrange an interview for a candidate.
- `POST /recruiting/offers/generate`: Create a job offer for a selected candidate.
- `GET /recruiting/analytics/pipeline`: Retrieve recruitment funnel metrics.

## Inter-service Interaction

- Emits `Candidate_Hired` event to `HCM` to automatically create an employee record.
- Queries `HCM Service` for department and position details for new requisitions.
- Emits `Background_Check_Requested` to external security providers.
- Queries `Identity Service` to create system access for hiring managers and interviewers.

## Key Logic

- **AI Candidate Matching:** Ranking candidates based on how well their skills/experience align with the job description.
- **Automated Screening:** Filtering out candidates who do not meet mandatory requirements (e.g., specific certifications).
- **Offer Approval Workflow:** Routing high-salary offers for executive or compensation-team approval.
- **Candidate Communication:** Automating status updates and interview reminders via email/SMS.
- **Internal Mobility:** Prioritizing internal employee applications via the "Opportunity Marketplace."
