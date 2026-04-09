# Partner Relationship Management (PRM) Service Specification

## Overview

The Partner Relationship Management (PRM) service manages the relationship between an organization and its channel partners (resellers, distributors, etc.). This aligns with Oracle Fusion Cloud Partner Relationship Management.

## Data Model (SQLite)

- `partner_accounts`: id (UUID), account_id (link to CDM), tier (GOLD/SILVER/BRONZE), status (ACTIVE/ONBOARDING/INACTIVE).
- `partner_programs`: id (UUID), name, status (ACTIVE/ARCHIVED), benefits (JSON), eligibility_criteria (JSON).
- `partner_leads`: id (UUID), lead_id (link to CRM), partner_id, status (ASSIGNED/ACCEPTED/REJECTED/EXPIRED), assign_date.
- `partner_opportunities`: id (UUID), opportunity_id (link to CRM), partner_id, deal_registration_id, status.
- `deal_registrations`: id (UUID), partner_id, customer_id, product_id, estimated_value, status (PENDING/APPROVED/REJECTED), expiration_date.
- `partner_funds`: id (UUID), partner_id, type (MDF/COOP), available_amount, currency, status.

## API Endpoints (Axum/gRPC)

- `POST /prm/partners/onboard`: Register a new channel partner.
- `POST /prm/leads/distribute`: Assign a lead to a specific partner based on geography or product expertise.
- `POST /prm/deals/register`: Record a new deal registration from a partner to prevent channel conflict.
- `GET /prm/funds/{partner_id}`: Retrieve current Market Development Funds (MDF) balance for a partner.
- `POST /prm/deals/approve`: Manager approval for a partner's deal registration.
- `GET /prm/portal/dashboard`: Retrieve key metrics for a specific partner.

## Inter-service Interaction

- Queries `CRM Service` for lead and opportunity metadata.
- Emits `Lead_Assigned` event to the partner via a portal or email.
- Receives `Order_Placed` from `Order Management` to trigger deal registration settlement.
- Queries `CDM Service` for master account and contact data.
- Emits `Partner_Award_Calculated` to `Incentive Compensation`.

## Key Logic

- **Deal Registration:** Preventing channel conflict by giving the first partner to register a deal a protected period or a higher discount.
- **Lead Distribution:** Automatically routing leads to partners based on criteria like location, industry, or partner tier.
- **MDF Management:** Managing funds for partner-led marketing activities (MDF) and co-op marketing budgets.
- **Channel Conflict Resolution:** Providing tools for managers to resolve cases where multiple partners or internal sales are pursuing the same customer.
- **Tiered Benefits:** Automatically applying different discount levels or lead types based on the partner's performance tier.
- **Partner Portal Integration:** Providing a secure workspace for partners to access training, deal registration, and sales collateral.
