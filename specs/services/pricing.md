# Pricing Service Specification

## Overview

The Pricing service provides a centralized engine for calculating item prices, discounts, and surcharges across all sales and service channels. It handles complex pricing strategies, including multi-currency, tiered pricing, and attribute-based rules. This aligns with Oracle Fusion Cloud Pricing (SCM).

## Data Model (SQLite)

- `pricing_strategies`: id (UUID), name, status (ACTIVE/DRAFT), currency, pricing_method (STATIC/DYNAMIC).
- `price_lists_central`: id (UUID), strategy_id, name, effective_date, expiration_date.
- `price_list_items`: id (UUID), price_list_id, item_id, base_price, status.
- `pricing_rules_engine`: id (UUID), strategy_id, rule_type (DISCOUNT/SURCHARGE/FREIGHT), criteria (JSON), adjustment_type (PERCENT/AMOUNT), adjustment_value.
- `tier_definitions`: id (UUID), name, min_qty, max_qty, unit_price.
- `attribute_pricing`: id (UUID), item_id, attribute_name, attribute_value, price_impact (ADD/OVERRIDE).

## API Endpoints (Axum/gRPC)

- `POST /pricing/calculate`: Execute the pricing engine for a set of items and customer context to retrieve the final net price.
- `GET /pricing/price-lists/search`: Search for items within active price lists.
- `POST /pricing/strategies/assign`: Assign a pricing strategy to a customer or business unit.
- `GET /pricing/tiers/{item_id}`: Retrieve volume-based pricing tiers for a specific item.
- `POST /pricing/rules/validate`: Ensure a set of pricing rules doesn't lead to negative margins or conflicting discounts.

## Inter-service Interaction

- Queries `Order Management / Commerce / CPQ` to provide real-time pricing during the quoting and ordering process.
- Queries `CRM Service` for customer tier and agreement details to inform pricing strategy selection.
- Receives `Item_Created` from `Product Hub` to initialize prices for new products.
- Queries `Maintenance Service` for billable labor rates and parts pricing.
- Emits `Price_Changed` event to `Marketing` for promotion alignment.

## Key Logic

- **Pricing Totals Calculation:** Summing up base price, applying discounts in a specific sequence (Cascading), and adding surcharges/taxes.
- **Dynamic Segment Pricing:** Automatically adjusting prices based on customer location, industry, or historical buying volume.
- **Matrix Pricing:** Calculating prices based on multiple product attributes (e.g., "Price = Base + [Color Premium] * [Size Factor]").
- **Volume Tiering:** Automatically reducing the unit price as the quantity ordered increases.
- **Currency Conversion:** Translating prices from a base price list into the customer's local currency using real-time rates.
- **Floor Price Enforcement:** Ensuring that sales reps cannot apply discounts that drop the price below a pre-set minimum (floor).
