# Common Types Specification

## Shared Data Structures

These types should be defined in a `common` crate and shared across all microservices and the frontend.

### Money

- **Representation:** `struct Money { amount: i64, currency: String }`
- **Units:** Minor units (cents for USD, cents for EUR).
- **Serialization:** String representation (e.g., "100.00 USD") or as a JSON object with integer and currency.

### Address

- **Fields:** line1, line2, city, state, postal_code, country_code (ISO 3166-1 alpha-2).

### Date/Time

- Use `chrono::DateTime<Utc>` for all timestamps.
- Use `chrono::NaiveDate` for business dates (e.g., Invoice Date).

### Quantity

- **Representation:** `struct Quantity { value: f64, unit_of_measure: String }`
- **UOM Example:** 'EA' (Each), 'KG', 'LITRE', 'HOUR'.

### Weight

- **Representation:** `struct Weight { value: f64, unit: String }`
- **Unit Example:** 'KG', 'LB'.

### Volume

- **Representation:** `struct Volume { value: f64, unit: String }`
- **Unit Example:** 'M3' (Cubic Meter), 'LITRE'.

### Common Error Codes

- `UNAUTHORIZED`: 401
- `FORBIDDEN`: 403
- `NOT_FOUND`: 404
- `BAD_REQUEST`: 400
- `INTERNAL_SERVER_ERROR`: 500
- `CONFLICT`: 409 (e.g., duplicate unique keys).
- `VALIDATION_ERROR`: Detailed error map for form validation.

## Status Enums

- **Workflow Status:** `DRAFT`, `PENDING`, `APPROVED`, `REJECTED`, `CLOSED`, `CANCELLED`.
- **Payment Status:** `UNPAID`, `PARTIALLY_PAID`, `PAID`, `OVERDUE`.

## Pagination

- **Request:** `page`, `page_size`, `sort_by`, `sort_order`.
- **Response:** `items`, `total_count`, `total_pages`, `current_page`.
