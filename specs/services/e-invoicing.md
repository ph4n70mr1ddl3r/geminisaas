# Global E-Invoicing & B2B Network Service Specification

## Overview

The Global E-Invoicing & B2B Network service manages the digital exchange of procurement and financial documents (Invoices, POs, ASNs) with trading partners and government tax authorities. It ensures compliance with global e-invoicing mandates (e.g., Peppol, SDIC). This aligns with Oracle Business Network (OBN) and Global E-Invoicing.

## Data Model (SQLite)

- `b2b_partner_network`: id (UUID), partner_name, network_id (e.g., Peppol ID), status (CONNECTED/PENDING), protocols (AS2/SFTP/REST).
- `e_invoice_headers`: id (UUID), source_doc_id (link to AP/AR), tax_authority_ack_id, status (SENT/ACCEPTED/REJECTED/VALIDATED), digital_signature.
- `tax_authority_endpoints`: id (UUID), country_code, service_url, certificate_id, validation_rules_json (JSON).
- `document_transformation_maps`: id (UUID), source_format (UBL/EDIFACT/JSON), target_format, version.
- `b2b_message_logs`: id (UUID), direction (INBOUND/OUTBOUND), message_type (INVOICE/PO), payload_url, status.
- `e_invoice_compliance_status`: id (UUID), document_id, clearance_timestamp, legal_archive_url.

## API Endpoints (Axum/gRPC)

- `POST /e-invoicing/send`: Dispatch a digital invoice to a partner or tax authority.
- `POST /e-invoicing/receive`: Ingest an inbound digital document from the B2B network.
- `GET /e-invoicing/status/{doc_id}`: Retrieve the real-time clearance status from a government portal.
- `POST /e-invoicing/partners/link`: Discover and link with a partner on the global business network.
- `GET /e-invoicing/compliance/report`: View audit logs for tax-compliant digital document exchange.
- `POST /e-invoicing/validate`: Run pre-submission checks against country-specific tax rules.

## Inter-service Interaction

- Receives `Invoice_Finalized` from `Financials (AR)` to trigger outbound clearance.
- Emits `Invoice_Received` event to `Financials (AP)` to automate voucher creation.
- Queries `Tax Service` for tax registration numbers and digital certificates.
- Emits `Clearance_Failed_Alert` to the billing or accounts payable team.
- Queries `Identity Service` for signing authority credentials.

## Key Logic

- **Clearance Model Support:** Automatically transmitting invoices to government portals for "Clearance" before they reach the customer (required in Italy, Brazil, India, etc.).
- **Digital Signatures:** Applying legally binding XAdES or PAdES signatures to documents using HSM-stored certificates.
- **Protocol Translation:** Seamlessly converting between different business languages (e.g., mapping an internal JSON to a Peppol UBL XML).
- **Auto-Provisioning:** Automatically finding and connecting with suppliers on the network based on their tax ID.
- **Legal Archiving:** Ensuring that digital invoices are stored in a tamper-proof manner for the duration required by local laws (e.g., 10 years).
- **Business Rule Validation:** Enforcing that mandatory fields (e.g., QR codes, specific tax codes) are present before transmission.
