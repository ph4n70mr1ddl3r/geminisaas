# Communications Launch Experience Service Specification

## Overview

The Communications Launch Experience service provides a unified product catalog and offer management system for the telecommunications industry. It enables rapid launch of complex bundles (voice, data, hardware, and content) across digital and physical channels. This aligns with Oracle Communications Launch Experience.

## Data Model (SQLite)

- `comm_product_catalog`: id (UUID), name, status (ACTIVE/RETIRED), type (OFFER/COMPONENT/RESOURCE), effective_date.
- `comm_offers`: id (UUID), catalog_id, name, bundle_type (TRIPLE_PLAY/SINGLE), pricing_structure (JSON), validity_period.
- `comm_promotion_codes`: id (UUID), offer_id, code, discount_pct, start_date, end_date, target_segment.
- `comm_eligibility_rules`: id (UUID), offer_id, criteria (JSON - e.g., location, existing services, credit score).
- `comm_order_enrichment`: id (UUID), source_order_id, attribute_name, attribute_value, mandatory (BOOL).
- `comm_launch_campaigns`: id (UUID), name, status (DRAFT/LIVE), launch_date, total_conversions.

## API Endpoints (Axum/gRPC)

- `POST /comm-launch/catalog/products`: Define a new product or service component in the catalog.
- `POST /comm-launch/offers/create`: Bundle components into a customer-facing offer with pricing.
- `GET /comm-launch/offers/eligible`: Retrieve available offers for a specific customer based on their context.
- `POST /comm-launch/campaigns/publish`: Push an offer to all digital storefronts and POS systems simultaneously.
- `GET /comm-launch/catalog/search`: Search the unified catalog for products or resources.
- `POST /comm-launch/orders/validate`: Ensure a configured bundle is technically and financially valid before submission.

## Inter-service Interaction

- Queries `Commerce Service` to display and sell communications offers on the digital storefront.
- Emits `Provisioning_Required` event to `Logistics/Manufacturing` for hardware fulfillment (e.g., SIM cards, routers).
- Queries `Subscription Management` for recurring billing structures and terms.
- Emits `Offer_Launch_Notification` to `Marketing` for targeted campaign execution.
- Queries `Financials` for regional tax and VAT application to complex bundles.

## Key Logic

- **Bundling Complexity:** Managing rules for "Any 3 for $X" or "Add Content for 50% Off" across different product types.
- **Telescopic Pricing:** Automatically adjusting prices based on contract duration or customer tier.
- **Compatibility Rules:** Ensuring that a high-speed data plan is only offered if the customer has a compatible modem or service area.
- **Time-to-Market Optimization:** Using templates to launch new promotional offers in minutes rather than weeks.
- **Channel Specificity:** Allowing an offer to have different pricing or attributes depending on whether it's sold via App, Web, or In-Store.
- **Resource Linking:** Connecting logical services (e.g., a 5G data plan) to physical inventory requirements.
