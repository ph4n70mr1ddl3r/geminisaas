# Behavioral Intelligence (Oracle Infinity) Service Specification

## Overview

The Behavioral Intelligence service captures and analyzes real-time customer behavior across digital channels (Web, Mobile, App) to trigger personalized experiences and improve conversion. This aligns with Oracle Infinity.

## Data Model (SQLite)

- `digital_streams`: id (UUID), source_name (WEB/IOS/ANDROID), tracking_id, status (ACTIVE/PAUSED).
- `behavioral_events`: id (UUID), stream_id, user_id (External/CDM), event_type (CLICK/SCROLL/VIEW/ADD_TO_CART), properties_json (JSON), timestamp.
- `real_time_scenarios`: id (UUID), name, trigger_condition (JSON - e.g., 'Basket Abandonment > $100'), target_action (JSON).
- `behavioral_segments`: id (UUID), name, definition_json, member_count, last_refreshed.
- `conversion_tunnels`: id (UUID), name, steps_json (JSON), drop_off_rates (JSON).
- `behavioral_heatmaps`: id (UUID), page_url, click_density_json, scroll_depth_avg.

## API Endpoints (Axum/gRPC)

- `POST /infinity/events/collect`: Ingest a high-volume stream of behavioral events from digital properties.
- `GET /infinity/segments/active`: Retrieve a list of segments a specific user currently belongs to.
- `POST /infinity/scenarios/deploy`: Activate a real-time behavioral trigger (e.g., show a popup if a user is about to leave).
- `GET /infinity/analytics/conversion`: Retrieve detailed funnel analysis for a specific conversion path.
- `POST /infinity/identity/stitch`: Link anonymous behavioral data to a known CDM customer record.
- `GET /infinity/real-time/dashboard`: View a live stream of user interactions and conversions.

## Inter-service Interaction

- Emits `High_Intent_Signal` event to `CRM` when a user shows strong buying behavior.
- Queries `CDM Service` to enrich behavioral events with customer profile data.
- Emits `Abandonment_Detected` event to `Marketing` for immediate email/SMS remarketing.
- Receives `Catalog_Data` from `Commerce` to provide context to product-related events.
- Queries `Unity CDP` for cross-channel customer orchestration.

## Key Logic

- **Identity Stitching:** Automatically linking behavioral events from different sessions and devices to a single "Golden Record" in CDM.
- **Real-Time Triggering:** Executing personalized actions (e.g., a discount offer) within milliseconds of a behavioral trigger.
- **Session Replay:** Providing the ability to "watch" a user's session to identify friction points in the UI.
- **Predictive Churn:** Identifying behavioral patterns that suggest a user is likely to abandon a process or service.
- **Unified Tagging:** Managing a single tracking tag across all digital properties to simplify data collection.
- **Behavioral Attribution:** Assigning value to specific digital interactions that led to a conversion.
