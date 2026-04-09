# Permitting & Licensing (Public Sector) Service Specification

## Overview

The Permitting & Licensing service provides specialized capabilities for government agencies to manage land use, building permits, business licenses, and professional certifications. This aligns with Oracle Fusion Cloud Permitting and Licensing (OPAL).

## Data Model (SQLite)

- `properties_gis`: id (UUID), address, parcel_number, owner_id, location_gis (JSON), zoning_type.
- `permit_applications`: id (UUID), type (BUILDING/ELECTRICAL/ZONING), applicant_id, property_id, status (SUBMITTED/REVIEW/INSPECTION/ISSUED), date_applied.
- `business_licenses`: id (UUID), business_name, license_type, status (ACTIVE/EXPIRED/RENEWAL_PENDING), effective_date, expiry_date.
- `permit_fees`: id (UUID), application_id, fee_type, amount, status (PENDING/PAID/PARTIAL).
- `inspections_field`: id (UUID), application_id, inspector_id, scheduled_date, status (PASS/FAIL), comments (JSON).
- `contractor_registry`: id (UUID), name, license_number, status (QUALIFIED/DEBARRED), insurance_expiry.

## API Endpoints (Axum/gRPC)

- `POST /permitting/apply`: Submit a new permit or license application.
- `GET /permitting/property/{address}`: Retrieve property and zoning data from GIS (e.g., ESRI integration).
- `POST /permitting/inspections/record`: Log findings from a field inspection.
- `GET /permitting/fees/calculate`: Run the fee engine based on permit type and project value.
- `POST /permitting/renew`: Initiate an automated renewal for a business license.
- `GET /permitting/public-portal/search`: Allow citizens to search for active permits in their area.

## Inter-service Interaction

- Queries `Financials Service` for fee payments and general ledger recording.
- Emits `Safety_Violation_Detected` to code enforcement teams.
- Receives `Identity_Verified` from `Identity Service` for citizen and contractor logins.
- Queries `Intelligent Advisor` for complex, rule-based application screening.
- Emits `License_Expired_Alert` to business owners via `Digital Assistant`.

## Key Logic

- **GIS-First Workflows:** Automatically pre-populating applications based on the physical property parcel selected from a map.
- **Dynamic Fee Engine:** Calculating complex multi-item fees based on project value, square footage, and jurisdiction rules.
- **Automated Renewals:** Identifying business licenses nearing expiration and automatically generating renewal invoices.
- **Partial Payment Support:** Allowing applicants to pay fees in installments while maintaining a real-time accounting balance.
- **Inspector Routing:** Optimizing the daily route for field inspectors based on geography and appointment priority.
- **Public Transparency:** Providing a citizen-facing portal for status tracking and public record searches.
