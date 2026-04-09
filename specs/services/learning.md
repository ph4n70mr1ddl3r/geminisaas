# Learning Service Specification

## Overview

The Learning service manages employee training, courses, certifications, and compliance tracking. This aligns with Oracle Fusion Cloud Learning.

## Data Model (SQLite)

- `learning_items`: id (UUID), title, description, type (COURSE/VIDEO/WEBINAR/IN_PERSON), duration_min, status (ACTIVE/ARCHIVED).
- `learning_offerings`: id (UUID), learning_item_id, delivery_mode, start_date, end_date, instructor_id, capacity.
- `learning_enrollments`: id (UUID), offering_id, employee_id, status (ENROLLED/IN_PROGRESS/COMPLETED/WAITLIST), completion_date, score.
- `learning_certifications`: id (UUID), name, validity_period_months, status, required_courses (JSON).
- `learning_completions`: id (UUID), certification_id, employee_id, expiry_date, status (ACTIVE/EXPIRED).
- `learning_comments`: id (UUID), learning_item_id, employee_id, text, rating (1-5), date.

## API Endpoints (Axum/gRPC)

- `GET /learning/catalog`: Search for available courses and training materials.
- `POST /learning/enroll`: Enroll an employee in a course or offering.
- `GET /learning/my-learning`: List assigned and optional training for the current user.
- `POST /learning/complete`: Record successful completion of a course or exam.
- `GET /learning/certifications/{emp_id}`: View active certifications for an employee.
- `POST /learning/recommend`: Suggest a learning item to a peer or direct report.

## Inter-service Interaction

- Queries `HCM Service` for employee skills and job roles to recommend relevant training.
- Emits `Certification_Expired` to `Workforce Health & Safety` for compliance-related training (e.g., OSHA).
- Receives `Employee_Hired` from `Recruiting/HCM` to assign initial onboarding training.
- Emits `Skill_Acquired` to `HCM` talent profile upon course completion.
- Queries `Identity Service` to verify instructor permissions.

## Key Logic

- **Compliance Automation:** Automatically re-assigning mandatory training every 12 months based on certification expiry.
- **Skill-Gap Analysis:** Recommending courses based on the delta between an employee's current skills and their job requirements.
- **Prerequisite Enforcement:** Ensuring learners have completed "Level 1" before enrolling in "Level 2."
- **Social Learning:** Enabling employees to rate, review, and share training content with their network.
- **Micro-Learning:** Supporting short-form content consumption (e.g., 5-minute videos) with progress tracking.
