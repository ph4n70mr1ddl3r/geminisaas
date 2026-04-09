# Supplier Portal Service Specification

## Overview

The Supplier Portal service provides a secure, collaborative environment for vendors to interact with the ERP. It handles supplier registration, bidding on sourcing events, and viewing POs/Invoices. This aligns with Oracle Fusion Cloud Supplier Portal and Sourcing.

## Data Model (SQLite)

- `supplier_registrations`: id (UUID), company_name, tax_id, contact_person, status (PENDING/APPROVED/REJECTED).
- `sourcing_events` (RFQ): id (UUID), title, end_date, requirements_document, status (ACTIVE/CLOSED).
- `supplier_bids`: id (UUID), event_id, vendor_id, price_quote, terms_and_conditions.
- `supplier_notices`: id (UUID), vendor_id, message, priority, read_status.
- `supplier_qualifications`: id (UUID), vendor_id, qualification_type (ISO-9001/SECURITY/DIVERSITY), expiry_date.

## API Endpoints (Axum/gRPC)

- `POST /supplier-portal/register`: Public endpoint for new vendors to apply.
- `GET /supplier-portal/pos`: Vendor view of their active Purchase Orders.
- `POST /supplier-portal/bids`: Submit a bid for an active sourcing event.
- `POST /supplier-portal/asn`: Submit an Advance Shipping Notice (ASN) to notify of an incoming delivery.
- `GET /supplier-portal/invoices`: View payment status of submitted invoices.

## Inter-service Interaction

- Queries `Procurement` for POs and vendor master data.
- Emits `Supplier_Approved` to `Procurement` to create a new record in the vendor master.
- Emits `ASN_Received` to `Logistics` to prepare for warehouse receiving.
- Queries `Financials` for invoice payment statuses.

## Key Logic

- **Self-Service Maintenance:** Allowing vendors to update their own contact information and bank details (subject to approval).
- **Bid Evaluation:** Scoring and comparing supplier bids based on price, quality, and lead time.
- **Advance Shipping Notices (ASN):** Automating the receiving process by having suppliers provide shipment details in advance.
- **Qualification Tracking:** Automatically notifying suppliers when their certifications (e.g., insurance, ISO) are about to expire.
