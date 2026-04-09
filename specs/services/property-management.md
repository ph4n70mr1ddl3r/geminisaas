# Property & Real Estate Management Service Specification

## Overview

The Property & Real Estate Management service manages the lifecycle of physical properties, including lease administration, space management, and real estate portfolio tracking. This aligns with Oracle Property Management (ERP).

## Data Model (SQLite)

- `property_portfolio`: id (UUID), name, type (OFFICE/RETAIL/WAREHOUSE/LAND), status, location_address, total_area_sqft.
- `property_units`: id (UUID), property_id, unit_number, floor, area_sqft, status (VACANT/LEASED/MAINTENANCE).
- `real_estate_leases`: id (UUID), property_id, unit_id, tenant_id (link to CDM), start_date, end_date, lease_type (GROSS/NET/PERCENTAGE).
- `lease_payment_terms`: id (UUID), lease_id, payment_type (RENT/UTILITY/TAX), amount, frequency (MONTHLY/QUARTERLY), escalation_pct.
- `space_occupancy`: id (UUID), unit_id, occupant_id (link to HCM/External), occupancy_type, start_date.
- `property_operating_costs`: id (UUID), property_id, cost_type (MAINTENANCE/INSURANCE/TAX), amount, date.

## API Endpoints (Axum/gRPC)

- `POST /property-mgmt/properties`: Add a new property to the corporate portfolio.
- `POST /property-mgmt/leases/create`: Record a new lease agreement for a property unit.
- `GET /property-mgmt/portfolio/occupancy`: Retrieve real-time occupancy rates and vacancy alerts.
- `POST /property-mgmt/billing/generate`: Generate rent and utility invoices for tenants.
- `POST /property-mgmt/leases/escalate`: Execute automated rent increases based on lease terms.
- `GET /property-mgmt/analytics/tco`: Analyze the Total Cost of Ownership for a property.

## Inter-service Interaction

- Queries `Lease Accounting` for financial valuation and compliance (IFRS 16/ASC 842).
- Emits `Property_Invoice_Generated` event to `Financials` (AR) or `Collections`.
- Emits `Maintenance_Required` event to `Maintenance Service` for facility repairs.
- Queries `Utilities Management` for sub-metered utility consumption by tenants.
- Emits `Asset_Capitalized` to `Fixed Assets` when a property improvement is completed.

## Key Logic

- **Lease Administration:** Managing the complex lifecycle of a lease, including options to renew, terminate, or expand.
- **Rent Escalation Engine:** Automatically calculating new rent amounts based on CPI, fixed percentages, or stepped increases.
- **Space Management:** Optimizing the utilization of desks, offices, and common areas.
- **Common Area Maintenance (CAM) Reconciliation:** Calculating and distributing shared property costs across multiple tenants.
- **Percentage Rent:** Calculating variable rent amounts based on a tenant's actual sales volume (common in Retail).
- **Critical Date Alerts:** Proactively notifying managers of upcoming lease expirations, break options, or insurance renewals.
