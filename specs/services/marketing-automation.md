# Marketing Automation (Eloqua/Responsys) Service Specification

## Overview

The Marketing Automation service provides advanced multi-channel campaign orchestration, lead nurturing, and behavioral triggering for B2B and B2C marketing. This aligns with Oracle Eloqua and Oracle Responsys.

## Data Model (SQLite)

- `orchestration_canvases`: id (UUID), name, status (DRAFT/ACTIVE), type (B2B/B2C), steps_json (JSON).
- `automation_triggers`: id (UUID), canvas_id, event_type (FORM_SUBMIT/EMAIL_CLICK/WEB_VISIT), filter_criteria (JSON).
- `multi_channel_content`: id (UUID), campaign_id, channel (EMAIL/SMS/PUSH/SOCIAL), content_json (JSON), status.
- `lead_nurture_streams`: id (UUID), name, segment_id, sequence_json (JSON), last_processed.
- `campaign_governance`: id (UUID), campaign_id, approval_status, fatigue_limit_check (BOOL), compliance_tags (JSON).
- `marketing_performance_ledger`: id (UUID), campaign_id, channel, period_id, send_count, click_count, conversion_count, revenue_attributed.

## API Endpoints (Axum/gRPC)

- `POST /marketing-automation/canvas`: Create a visual campaign orchestration workflow.
- `POST /marketing-automation/trigger`: Manually fire an event to inject a customer into an automation canvas.
- `GET /marketing-automation/nurture/status`: Monitor the progress of leads through a multi-step nurture program.
- `POST /marketing-automation/content/personalize`: Retrieve AI-personalized content for a specific customer and channel.
- `GET /marketing-automation/analytics/attribution`: View cross-channel attribution reports for marketing-driven revenue.
- `POST /marketing-automation/fatigue/check`: Validate if a customer has received too many messages within a time window.

## Inter-service Interaction

- Receives `Behavioral_Event` from `Behavioral Intelligence` to trigger real-time campaigns.
- Emits `Qualified_Lead` to `CRM` after a nurture sequence completes successfully.
- Queries `CDM Service` for deep customer attributes used in complex personalization rules.
- Queries `Commerce` for product catalog data to include in abandoned cart emails.
- Emits `Marketing_Activity_Logged` to `Analytics Service` for long-term ROI reporting.

## Key Logic

- **Visual Orchestration:** Providing a drag-and-drop canvas for defining complex customer journeys with conditional logic.
- **Dynamic Content:** Automatically substituting images, text, and offers based on the recipient's profile and behavior.
- **Fatigue Management:** Preventing "over-marketing" by enforcing frequency caps across all communication channels.
- **Lead Nurturing:** Gradually educating and qualifying leads through a series of automated interactions over weeks or months.
- **Program Replication:** Allowing successful campaign templates to be quickly cloned and localized for different regions.
- **Global Compliance:** Automatically enforcing "Opt-Out" and "Do Not Contact" rules across all countries and regulations (GDPR/CCPA).
