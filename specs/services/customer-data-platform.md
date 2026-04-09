# Customer Data Platform (Unity) Service Specification

## Overview

The Customer Data Platform (Unity) service provides a centralized hub for unifying customer data from all sources (ERP, CRM, Behavioral, Social, Offline). It uses AI to create high-definition customer profiles and orchestrate personalized experiences across channels. This aligns with Oracle Unity CDP.

## Data Model (SQLite/NoSQL Proxy)

- `unified_customer_profiles`: id (UUID), master_customer_id (link to CDM), unified_attributes (JSON), lifetime_value, churn_probability.
- `customer_identity_map`: id (UUID), identifier_type (EMAIL/COOKIE/PHONE), identifier_value, master_customer_id.
- `customer_segments_cdp`: id (UUID), name, definition_json (JSON), member_count, update_frequency.
- `customer_experience_events`: id (UUID), customer_id, channel (WEB/MOBILE/STORE), interaction_type, sentiment_score, timestamp.
- `cdp_data_sources`: id (UUID), name, status (CONNECTED/PAUSED), last_ingest_time, record_count.
- `orchestration_workflows_cdp`: id (UUID), name, trigger_segment_id, action_json (JSON), status.

## API Endpoints (Axum/gRPC)

- `POST /cdp/ingest`: Ingest customer data from a source system (e.g., a file upload or streaming API).
- `GET /cdp/profile/{id}`: Retrieve the high-definition, unified profile for a customer.
- `POST /cdp/segments/calculate`: Run the segment builder to identify customers matching specific criteria.
- `GET /cdp/recommendations/personalized`: Retrieve AI-suggested next-best-actions or products for a customer.
- `POST /cdp/identity/link`: Manually or automatically link multiple identities to a single master profile.
- `GET /cdp/analytics/journey`: Retrieve a visualization of the cross-channel customer journey.

## Inter-service Interaction

- Receives `Behavioral_Event` from `Behavioral Intelligence (Infinity)` for real-time profile updates.
- Emits `Segment_Updated` event to `Marketing Automation` to trigger targeted campaigns.
- Queries `CDM Service` for foundational master record data.
- Emits `Personalized_Offer` to `Commerce / CRM` for real-time display.
- Queries `Loyalty Service` for member tier and points balance.

## Key Logic

- **Identity Resolution:** Probabilistically and deterministically linking disparate data points into a single customer ID.
- **High-Definition Profiling:** Aggregating thousands of attributes (e.g., "Frequent Traveler", "Loves Blue") from all interactions.
- **Real-Time Orchestration:** Triggering an action in one system based on an event in another (e.g., if a user abandons a cart on the web, send a personalized discount via the App).
- **Predictive Modeling:** Using ML to calculate individual scores for churn, sentiment, and propensity-to-buy.
- **Consent Management:** Ensuring that all data usage respects customer privacy preferences and global regulations (GDPR/CCPA).
- **Data Enrichment:** Automatically pulling in third-party data (e.g., demographics, weather) to enhance profiles.
