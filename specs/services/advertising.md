# Advertising & Audience Intelligence Service Specification

## Overview

The Advertising & Audience Intelligence service provides capabilities for digital advertising measurement, contextual targeting, and audience segmentation using behavioral data. This aligns with Oracle Advertising (Moat, Grapeshot, and Unity CDP integration).

## Data Model (SQLite)

- `ad_campaigns_digital`: id (UUID), name, platform (GOOGLE/META/PROGRAMMATIC), status, budget, start_date, end_date.
- `audience_segments_ad`: id (UUID), name, criteria_json (JSON), member_count, platform_sync_status.
- `advertisement_measurement`: id (UUID), campaign_id, metric_name (VIEWABILITY/INVALID_TRAFFIC/ATTENTION), value, timestamp.
- `contextual_segments`: id (UUID), name, keywords (JSON), safety_score, category (HEALTH/FINANCE/NEWS).
- `audience_activation_logs`: id (UUID), segment_id, target_platform, records_synced, timestamp.
- `advertising_roi_ledger`: id (UUID), campaign_id, ad_spend, attributed_revenue (from Commerce), period_id.

## API Endpoints (Axum/gRPC)

- `POST /advertising/audiences/create`: Define an audience segment based on behavioral and CDM data.
- `POST /advertising/audiences/activate`: Push a segment to an external advertising platform (e.g., Facebook Ads).
- `GET /advertising/measurement/viewability`: Retrieve high-accuracy metrics on whether ads were actually seen by humans.
- `POST /advertising/contextual/analyze`: Retrieve contextual safety and relevance scores for a specific URL or content piece.
- `GET /advertising/analytics/attribution`: Calculate the "Brand Lift" or conversion rate driven by digital advertising.

## Inter-service Interaction

- Queries `Unity CDP / CDM` for unified customer profiles to build seed audiences.
- Receives `Conversion_Event` from `Commerce / CRM` to close the loop on ad attribution.
- Queries `Behavioral Intelligence (Infinity)` for real-time digital interaction data.
- Emits `Advertising_Expense` to `Financials` for budget tracking.
- Queries `Marketing` for overall campaign strategy and creative assets.

## Key Logic

- **Viewability & IVT Detection:** Using advanced telemetry (Moat) to filter out bot traffic and ensure ads are displayed in viewable areas.
- **Contextual Intelligence:** Analyzing page content (Grapeshot) to ensure ads appear alongside relevant and "brand-safe" content.
- **Lookalike Modeling:** Identifying new potential customers who share behaviors with existing high-value customers.
- **Identity Resolution:** Bridging anonymous ad IDs (Cookies, MAIDs) with known customer records in a privacy-compliant manner.
- **Multi-Touch Attribution:** Determining the relative contribution of each ad interaction to a final sale.
- **Privacy Sandbox Integration:** Adapting to post-cookie tracking environments using API-based cohort analysis.
