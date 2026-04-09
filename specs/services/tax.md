# Tax Service Specification

## Overview

The Tax service handles global indirect tax calculation (Sales Tax, VAT, GST, etc.) for both payables and receivables. It integrates with external tax engines (e.g., Vertex, Avalara) or uses internal tax rules.

## Data Model (SQLite)

- `tax_regimes`: id (UUID), country_code, name, status (ACTIVE/INACTIVE).
- `tax_rates`: id (UUID), regime_id, tax_type (VAT/SALES_TAX/GST), percentage, effective_start, effective_end.
- `tax_exemptions`: id (UUID), customer_id, tax_type, reason, certificate_number.
- `tax_transactions`: id (UUID), source_id (Invoice/Order), type (PAYABLE/RECEIVABLE), tax_amount, taxable_basis, regime_id.
- `tax_jurisdictions`: id (UUID), regime_id, state/province, city, postal_code_range.

## API Endpoints (Axum/gRPC)

- `POST /tax/calculate`: Calculate tax for a specific transaction (takes line items and addresses).
- `POST /tax/exemptions`: Register a customer tax exemption certificate.
- `GET /tax/reports/liability`: Generate a report showing tax collected vs. tax paid for a period.
- `POST /tax/validate-id`: Validate a VAT/GST registration number for a vendor or customer.

## Inter-service Interaction

- Called by `Procurement` during invoice entry to calculate input tax.
- Called by `Order Management` during order creation to calculate output tax for the customer quote.
- Called by `Financials` during manual journal entry creation if tax-relevant accounts are used.
- Emits `Tax_Calculated` events to `Financials` for ledger posting.

## Key Logic

- **Nexus Determination:** Logic to determine which tax jurisdictions apply based on ship-to, ship-from, and point-of-sale addresses.
- **Tax Inclusion/Exclusion:** Handling prices that already include tax (VAT-style) vs. those that don't (US Sales Tax-style).
- **Reverse Charge:** Handling VAT reverse charge mechanisms for B2B transactions.
- **Rounding Rules:** Applying specific rounding logic (e.g., round to nearest cent) as required by local tax authorities.
