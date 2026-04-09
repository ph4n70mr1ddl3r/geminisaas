# Commerce Service Specification

## Overview

The Commerce service provides high-scale B2B and B2C digital storefront capabilities, managing catalogs, pricing, and shopping carts. This aligns with Oracle Fusion Cloud Commerce.

## Data Model (SQLite)

- `catalogs`: id (UUID), name, status (ACTIVE/DRAFT), language.
- `catalog_categories`: id (UUID), catalog_id, parent_id, name, description.
- `catalog_products`: id (UUID), catalog_id, item_id (link to Inventory), display_name, description, image_urls (JSON), status.
- `price_lists`: id (UUID), name, currency, effective_date, status.
- `price_list_entries`: id (UUID), price_list_id, product_id, price.
- `shopping_carts`: id (UUID), account_id (link to CDM), status (ACTIVE/ORDERED/ABANDONED), last_updated.
- `cart_items`: id (UUID), cart_id, product_id, quantity, unit_price.
- `promotions`: id (UUID), name, code, discount_pct, start_date, end_date.

## API Endpoints (Axum/gRPC)

- `GET /commerce/catalog/{id}/products`: Retrieve products for a digital storefront.
- `POST /commerce/cart`: Create or update a shopping cart.
- `GET /commerce/cart/{id}`: View cart contents.
- `POST /commerce/checkout`: Finalize a cart and initiate order creation.
- `POST /commerce/promotions/apply`: Apply a discount code to a cart.

## Inter-service Interaction

- Queries `Inventory Service` for real-time stock levels before display.
- Queries `CPQ Service` for complex product configuration and dynamic pricing.
- Emits `Cart_Checkout_Completed` event to `Order Management` to create a sales order.
- Queries `CDM Service` for customer profile and past purchase history to provide personalization.
- Emits `Cart_Abandoned` event to `Marketing` for follow-up campaigns.

## Key Logic

- **B2B Account Pricing:** Showing customer-specific prices based on pre-negotiated contracts.
- **Unified Cart:** Allowing customers to start a cart on mobile and finish it on a desktop.
- **Search & Discovery:** Integrating with full-text search engines (e.g., Meilisearch) for fast product lookup.
- **Tax Calculation:** Requesting real-time tax from `Tax Service` during checkout based on delivery address.
- **Shipping Estimates:** Requesting freight costs from `Logistics Service` based on cart weight and destination.
