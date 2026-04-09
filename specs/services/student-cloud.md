# Student Cloud Service Specification

## Overview

The Student Cloud service provides a complete student lifecycle management system, including admissions, financial aid, academics, and student finance. This aligns with Oracle Fusion Cloud Student Cloud.

## Data Model (SQLite)

- `student_records`: id (UUID), first_name, last_name, email, phone, status (PROSPECT/APPLIED/ENROLLED/ALUMNI), date_of_birth.
- `academic_programs`: id (UUID), name, status (ACTIVE/ARCHIVED), type (DEGREE/NON_DEGREE/CERTIFICATE), credits_required.
- `courses`: id (UUID), program_id, course_code, name, status, credits.
- `student_enrollments`: id (UUID), student_id, course_id, status (ENROLLED/IN_PROGRESS/COMPLETED/WITHDRAWN), grade, completion_date.
- `financial_aid_awards`: id (UUID), student_id, award_year, type (GRANT/LOAN/SCHOLARSHIP), status (PENDING/ACCEPTED/DISBURSED), amount.
- `student_finance_accounts`: id (UUID), student_id, balance, last_transaction_date, status.
- `course_planner_scenarios`: id (UUID), student_id, courses_planned (JSON), target_graduation_date.

## API Endpoints (Axum/gRPC)

- `POST /student-cloud/admissions`: Record a new student application.
- `GET /student-cloud/records/search`: Search for student records and academic history.
- `POST /student-cloud/enroll`: Enroll a student in a course or program.
- `GET /student-cloud/financial-aid/eligibility`: Retrieve current financial aid status and awards.
- `POST /student-cloud/finance/billing`: Generate a tuition or fee invoice for a student.
- `GET /student-cloud/academics/planner`: View a student's graduation progress and plan.

## Inter-service Interaction

- Queries `Financials Service` for student billing and payments.
- Emits `Award_Disbursed` event to `Financials` to update student finance accounts.
- Receives `Employee_Hired` from `Recruiting/HCM` for students who are also campus employees.
- Queries `Identity Service` for faculty and staff permissions.
- Emits `Degree_Conferred` event to `HCM` talent profile.
- Queries `Learning Service` for continuing education or non-degree courses.

## Key Logic

- **Financial Aid Automation:** Automatically determining aid eligibility based on student profile and award year rules.
- **Academic Progress Tracking:** Calculating GPA and monitoring completion of degree requirements (Degree Audit).
- **Lifelong Learning:** Supporting a "One-Student, Multiple-Programs" model where a student can return for multiple degrees or certificates over their lifetime.
- **Dynamic Billing:** Automatically calculating tuition and fees based on residency, program, and credit load.
- **Waitlist Management:** Automatically enrolling students from a waitlist when a course seat becomes available.
- **Degree Planning:** Simulating various paths to graduation based on course availability and prerequisites.
- **Award Verification:** Managing the verification process for financial aid to meet regulatory requirements (e.g., FAFSA).
