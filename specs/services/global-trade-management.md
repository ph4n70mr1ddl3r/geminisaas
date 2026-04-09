# Global Trade Management (GTM) Service Specification

## Overview

The Global Trade Management (GTM) service manages cross-border trade compliance, customs filing, and restricted party screening. This aligns with Oracle Fusion Cloud Global Trade Management.

## Data Model (SQLite)

- `trade_items`: id (UUID), item_id (link to Inventory), hts_code (Harmonized Tariff Schedule), eccn (Export Control Classification Number), country_of_origin.
- `restricted_parties`: id (UUID), name, address, source_list (OFAC/UN/EU), status (CLEARED/DENIED/PENDING).
- `trade_screenings`: id (UUID), partner_id, status (PASSED/FAILED), reason, timestamp.
- `customs_declarations`: id (UUID), shipment_id (link to Logistics), declaration_number, status (DRAFT/SUBMITTED/RELEASED), customs_value, duties_amount.
- `trade_licenses`: id (UUID), license_number, type (EXPORT/IMPORT), issuer, start_date, end_date, total_value_limit, remaining_value.
- `trade_incentives`: id (UUID), type (DUTY_DRAWBACK/FTA), amount_claimed, status.

## API Endpoints (Axum/gRPC)

- `POST /gtm/screen`: Screen a partner against restricted party lists.
- `GET /gtm/classification/{item_id}`: Retrieve trade classification and tariff codes for an item.
- `POST /gtm/declarations`: Generate a customs declaration for a shipment.
- `GET /gtm/licenses/valid`: Check for valid licenses for a specific trade transaction.
- `POST /gtm/compliance-check`: Perform a comprehensive trade compliance check (License + Screening).
- `GET /gtm/duty-calculator`: Estimate customs duties based on HTS code and value.

## Inter-service Interaction

- Receives `Shipment_Created` from `Logistics` to trigger customs filing.
- Queries `CRM/Procurement` for partner details to perform restricted party screening.
- Emits `Trade_Hold_Placed` to `Order Management` or `Logistics` if a compliance check fails.
- Emits `Duty_Paid` event to `Financials` for tax and duty recording.
- Queries `Inventory Service` for product weights, origins, and descriptions.

## Key Logic

- **Fuzzy Matching:** Using advanced text matching algorithms to identify potential restricted parties even with slight spelling variations.
- **Classification Engine:** Automatically suggesting HTS codes based on product descriptions and machine learning history.
- **License Determination:** Matching transaction details (Item, Country, Partner) against available export/import licenses.
- **Embargo Checks:** Blocking all transactions to/from countries with active trade sanctions.
- **Duty Drawback:** Identifying opportunities to reclaim paid duties for items that are subsequently re-exported.
