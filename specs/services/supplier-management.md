# Supplier Management & Incentives Service Specification

## Overview

The Supplier Management service handles the end-to-end lifecycle of vendors, including onboarding, qualification, risk assessment, and supplier incentive (rebate) management. This aligns with Oracle Fusion Cloud Supplier Management and Channel Revenue Management (Supplier).

## Data Model (SQLite)

- `supplier_master`: id (UUID), name, status (ACTIVE/PROSPECT/DEBARRED), risk_score, diversity_status, primary_contact_id.
- `supplier_qualifications`: id (UUID), supplier_id, type (FINANCIAL/ISO/SECURITY), score, expiry_date, status.
- `supplier_rebate_programs`: id (UUID), name, type (VOLUME/GROWTH/FLAT), start_date, end_date, criteria_json (JSON).
- `rebate_accruals_supplier`: id (UUID), program_id, source_po_id, amount_accrued, status (PENDING/CLAIMED).
- `supplier_claims_incentive`: id (UUID), program_id, amount_claimed, status (DRAFT/SUBMITTED/PAID), claim_date.
- `supplier_risk_incidents`: id (UUID), supplier_id, description, severity, source (EXTERNAL/INTERNAL), status.

## API Endpoints (Axum/gRPC)

- `POST /supplier-management/onboard`: Initiate the onboarding workflow for a new vendor.
- `POST /supplier-management/qualify`: Record the results of a supplier qualification assessment.
- `GET /supplier-management/risk/{id}`: Retrieve a comprehensive risk profile for a supplier.
- `POST /supplier-management/rebates/accrue`: Record potential rebate earnings based on purchase volume.
- `POST /supplier-management/claims/submit`: Generate a claim to a supplier for earned rebates or incentives.
- `GET /supplier-management/analytics/spend-diversity`: View spend metrics across diverse and local suppliers.

## Inter-service Interaction

- Queries `Procurement` for historical spend and active Purchase Orders.
- Emits `Supplier_Qualified` event to `Procurement` to enable purchasing.
- Emits `Debit_Memo_Requested` to `Financials` (AP) to collect earned rebates from a vendor.
- Queries `Risk Management` for external risk signals (e.g., news, credit scores).
- Emits `Onboarding_Complete` notification to the `Supplier Portal`.

## Key Logic

- **Automated Lifecycle Workflows:** Managing the transition from a "Prospective" vendor to a "Qualified" and "Active" one through multi-step approvals.
- **Volume Rebate Calculation:** Automatically tracking spend against tiered rebate targets (e.g., "Earn 2% back after $1M in purchases").
- **Supplier Risk Scoring:** Aggregating data from multiple sources (financials, quality, external signals) into a single risk index.
- **Diversity Tracking:** Monitoring spend against ESG and diversity goals (e.g., Minority-Owned, Women-Owned).
- **Certificate Management:** Proactively notifying vendors via the portal when their insurance or ISO certificates are nearing expiration.
- **Supplier Hub Synchronization:** Ensuring a single, clean supplier record across all regional ERP instances.
