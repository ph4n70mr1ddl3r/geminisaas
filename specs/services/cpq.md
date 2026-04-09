# CPQ (Configure, Price, Quote) Service Specification

## Overview

The CPQ service handles complex product configurations, pricing rules, and professional quote generation. This aligns with Oracle Fusion Cloud CPQ.

## Data Model (SQLite)

- `product_models`: id (UUID), name, base_price, configuration_rules (JSON).
- `product_attributes`: id (UUID), model_id, name, type (COLOR/SIZE/SPEC), options (JSON).
- `price_books`: id (UUID), name, currency, effective_date, status.
- `price_book_entries`: id (UUID), price_book_id, item_id, list_price, discount_rules (JSON).
- `quotes`: id (UUID), opportunity_id (link to CRM), customer_id, total_amount, status (DRAFT/APPROVED/SENT/ACCEPTED).
- `quote_lines`: id (UUID), quote_id, item_id, selected_attributes (JSON), unit_price, discount, net_price.
- `discount_approvals`: id (UUID), quote_id, approver_id, discount_pct, status.

## API Endpoints (Axum/gRPC)

- `POST /cpq/quotes`: Create a new quote for an opportunity.
- `GET /cpq/configurator/{model_id}`: Retrieve available configuration options for a product.
- `POST /cpq/quotes/{id}/calculate`: Run the pricing engine on a quote.
- `POST /cpq/quotes/{id}/approve`: Request approval for deep discounts.
- `GET /cpq/quotes/{id}/document`: Generate a PDF quote document.

## Inter-service Interaction

- Queries `CRM Service` for opportunity and customer details.
- Queries `Inventory Service` for item IDs and descriptions.
- Emits `Quote_Accepted` event to `Order Management` to create a sales order.
- Queries `Financials` for regional tax rates to apply to quotes.

## Key Logic

- **Constraint Engine:** Ensuring that selected product options are compatible (e.g., "Feature A requires Option B").
- **Tiered Pricing:** Automatically applying volume-based discounts or promotional pricing.
- **Approval Thresholds:** Logic for routing quotes for manager approval if the discount exceeds a certain percentage (e.g., >15%).
- **Subscription Support:** Handling recurring revenue pricing (MRR/ARR) and renewal uplift logic.
- **Quote-to-Order Conversion:** Seamlessly mapping configured items and agreed prices into the order management system.
