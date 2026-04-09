# Opportunity Marketplace Service Specification

## Overview

The Opportunity Marketplace service provides a central hub for internal mobility, enabling employees to discover and apply for full-time roles, short-term gigs, and volunteer opportunities. This aligns with Oracle Fusion Cloud HCM Opportunity Marketplace and Oracle ME.

## Data Model (SQLite)

- `marketplace_postings`: id (UUID), title, description, type (FULL_TIME/GIG/VOLUNTEER), status (ACTIVE/FILLED/CANCELLED), hiring_manager_id, start_date, end_date, skills_required (JSON).
- `gig_assignments`: id (UUID), posting_id, employee_id, status (PROPOSED/ACCEPTED/COMPLETED), actual_start_date, actual_end_date, effort_hours.
- `employee_marketplace_profiles`: id (UUID), employee_id, skills (JSON), interests (JSON), availability_pct.
- `marketplace_applications`: id (UUID), posting_id, employee_id, status (SUBMITTED/REVIEW/SELECTED/REJECTED), application_date.
- `gig_feedback`: id (UUID), assignment_id, reviewer_id, rating (1-5), comments.
- `marketplace_analytics`: id (UUID), period_id, total_gigs, total_mobility_events, skill_growth_index.

## API Endpoints (Axum/gRPC)

- `GET /marketplace/postings`: Search for internal opportunities based on skills and interests.
- `POST /marketplace/postings`: Create a new internal gig or posting.
- `POST /marketplace/apply`: Submit an application for an internal posting.
- `GET /marketplace/my-gigs`: View current and past gig assignments.
- `POST /marketplace/feedback`: Provide feedback on a completed gig.
- `GET /marketplace/recommended`: Retrieve personalized opportunity recommendations using AI.

## Inter-service Interaction

- Queries `HCM Service` for employee core data, skills, and current position.
- Emits `Gig_Completed` event to `HCM` and `Learning` to update employee skills and experience.
- Receives `Employee_Terminated` from `HCM` to automatically cancel active marketplace applications.
- Queries `Recruiting Service` for internal-only job requisitions.
- Emits `Assignment_Notification` to the employee and their current manager.

## Key Logic

- **AI Skill-Match Recommender:** Automatically matching employees to opportunities based on the delta between their current skills and the posting's requirements.
- **Gig Effort Management:** Tracking the time commitment for short-term gigs to ensure it doesn't conflict with the employee's primary role.
- **Internal Talent Visibility:** Allowing hiring managers to discover internal talent who have relevant skills but aren't actively searching.
- **Skill Growth Tracking:** Quantifying the skill development that occurs through gig participation.
- **Manager Approval Workflow:** Ensuring the employee's current manager is informed and/or approves of their participation in a gig.
- **Volunteer Integration:** Enabling the organization to promote social responsibility by posting volunteer events alongside professional opportunities.
