# Sourcing & Supplier Qualification Service Specification

## Overview

The Sourcing service manages the negotiation and bidding lifecycle, including RFQs (Request for Quotations), auctions, and supplier evaluations. This aligns with Oracle Fusion Cloud Sourcing and Supplier Qualification.

## Data Model (SQLite)

- `negotiations`: id (UUID), title, type (RFQ/AUCTION/RFI), status (DRAFT/OPEN/CLOSED/AWARDED), close_date, template_id.
- `negotiation_lines`: id (UUID), negotiation_id, item_id, quantity, start_price, target_price.
- `supplier_invitations`: id (UUID), negotiation_id, vendor_id, status (INVITED/ACCEPTED/DECLINED).
- `supplier_bids`: id (UUID), negotiation_id, vendor_id, bid_price, bid_rank, status (SUBMITTED/REJECTED/WINNING).
- `qualification_questionnaires`: id (UUID), vendor_id, type (FINANCIAL/ISO/SECURITY), score, status (QUALIFIED/NOT_QUALIFIED).
- `supplier_certs`: id (UUID), vendor_id, cert_name (ISO9001/SOC2), expiry_date, image_url.

## API Endpoints (Axum/gRPC)

- `POST /sourcing/negotiations`: Create a new RFQ or auction.
- `POST /sourcing/bids/submit`: Record a supplier's bid for a negotiation.
- `GET /sourcing/negotiations/{id}/rankings`: View the top bids by price and score.
- `POST /sourcing/award`: Select the winning supplier for a negotiation.
- `POST /sourcing/qualification/assess`: Complete a qualification questionnaire for a vendor.

## Inter-service Interaction

- Receives `Requisition_Approved` from `Procurement` to start a negotiation for high-value items.
- Emits `Supplier_Awarded` to `Procurement` to automatically generate a Purchase Order.
- Queries `Risk Management` for vendor risk scores during the award process.
- Emits `Vendor_Qualified` to `Procurement` to allow the vendor to receive future POs.

## Key Logic

- **Reverse Auctions:** Automatically ranking bids in real-time as suppliers lower their prices during a live auction.
- **Weighted Scoring:** Evaluating bids based on a mix of price (e.g., 70%) and supplier quality score (e.g., 30%).
- **Multi-Round Negotiations:** Supporting subsequent rounds of bidding to narrow down the vendor list.
- **Compliance Enforcement:** Automatically blocking awards to suppliers with expired or missing certifications.
- **Price Trend Analysis:** Comparing current bid prices against historical PO prices for the same items.
