# Marketing Service Specification

## Overview

The Marketing service manages multi-channel campaigns, lead generation, customer segmentation, and marketing analytics. It sits at the top of the funnel for CRM.

## Data Model (SQLite)

- `campaigns`: id (UUID), name, type (EMAIL/SOCIAL/WEB), status (DRAFT/ACTIVE/PAUSED/FINISHED), budget (DECIMAL), start_date, end_date.
- `segments`: id (UUID), name, filter_criteria (JSON), population_count.
- `marketing_leads`: id (UUID), campaign_id, contact_info, score (INTEGER), source.
- `content_assets`: id (UUID), name, type (IMAGE/VIDEO/TEMPLATE), storage_url.
- `interactions`: id (UUID), lead_id, type (OPEN/CLICK/CONVERT), timestamp.
- `social_integrations`: id (UUID), platform (LINKEDIN/TWITTER/META), access_token.

## API Endpoints (Axum/gRPC)

- `POST /marketing/campaign`: Create and launch marketing campaigns.
- `GET /marketing/segments`: Manage target customer segments.
- `POST /marketing/lead-gen`: Record a new lead from a marketing source.
- `GET /marketing/campaign-roi`: View return-on-investment metrics for active campaigns.
- `POST /marketing/segment-sync`: Update segment population based on CRM customer data.

## Inter-service Interaction

- Emits `Qualified_Lead` event to `CRM Service` when a marketing lead reaches a specific score.
- Queries `CRM Service` for existing customer data to build segments.
- Emits `Campaign_Expense` event to `Financials` for budget tracking.
- Queries `Analytics Service` for historical conversion data to optimize campaign targeting.

## Key Logic

- **Lead Scoring:** Automatically increment lead scores based on interaction frequency (e.g., +10 for click, +50 for form fill).
- **A/B Testing:** Compare the performance of two campaign variations (e.g., subject lines, creatives).
- **Campaign Attribution:** Track which marketing touchpoint led to a sale (First-touch vs. Last-touch).
- **Email Automation:** Schedule and send automated emails based on lead behavior or segment triggers.
