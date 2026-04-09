# Internal Communications (Oracle Communicate) Service Specification

## Overview

The Internal Communications service manages the creation, distribution, and tracking of organization-wide or targeted messages, newsletters, and events. This aligns with Oracle Communicate, part of the Oracle ME employee experience suite.

## Data Model (SQLite)

- `communication_campaigns`: id (UUID), title, status (DRAFT/SCHEDULED/SENT), type (NEWSLETTER/EVENT/URGENT), start_date, end_date.
- `communication_messages`: id (UUID), campaign_id, content_body (HTML/Markdown), sender_name, subject_line.
- `communication_audiences`: id (UUID), campaign_id, filter_criteria (JSON - e.g., 'All employees in UK'), population_count.
- `communication_interactions`: id (UUID), message_id, employee_id, action (OPEN/CLICK/RESPOND), timestamp.
- `communication_events`: id (UUID), campaign_id, event_name, location (VIRTUAL/PHYSICAL), start_time, end_time, capacity.
- `event_registrations`: id (UUID), event_id, employee_id, status (REGISTERED/ATTENDED/CANCELLED).

## API Endpoints (Axum/gRPC)

- `POST /communicate/campaigns`: Create a new internal communication campaign.
- `POST /communicate/audiences`: Define a target audience for a message based on HCM data.
- `POST /communicate/messages/send`: Dispatch a message via multiple channels (Email, SMS, App).
- `GET /communicate/analytics/reach`: Retrieve engagement metrics (open rates, click rates) for a campaign.
- `POST /communicate/events/register`: Enroll an employee in a company event or webinar.
- `GET /communicate/my-newsletters`: Retrieve personalized newsletter digests for the logged-in user.

## Inter-service Interaction

- Queries `HCM Service` for employee core data, location, and department to build audiences.
- Emits `New_Internal_Event` notification to employees via `Digital Assistant`.
- Queries `Journeys Service` to embed communication messages into specific employee journeys.
- Emits `Low_Engagement_Alert` to HR managers if critical messages are not being read.
- Queries `Identity Service` for employee email addresses and phone numbers.

## Key Logic

- **Precision Targeting:** Using complex HCM-based logic to ensure messages only reach the relevant people (e.g., "Managers with direct reports in France").
- **Multi-Channel Dispatch:** Automatically sending a single message across email, mobile push, and internal portals.
- **Newsletter Digests:** Aggregating multiple updates into a single, beautiful newsletter to reduce noise.
- **Event Attendance Tracking:** Managing RSVPs and recording actual attendance for company-wide meetings or training.
- **AI-Driven Personalization:** Suggesting internal events or news to employees based on their interests and career goals (from `Oracle Grow`).
- **Contextual Reach:** Displaying communications within the context of related tasks (e.g., a message about new benefits appearing on the Benefits page).
