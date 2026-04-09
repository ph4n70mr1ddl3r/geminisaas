# Tax Reporting Service Specification

## Overview

The Tax Reporting service manages global tax provisioning, country-by-country reporting (CbCR), and compliance with international tax regulations such as OECD Pillar Two. This aligns with Oracle Fusion Cloud Tax Reporting.

## Data Model (SQLite)

- `tax_provisions`: id (UUID), period_id, legal_entity_id, status (DRAFT/APPROVED/FINAL), effective_tax_rate, current_tax_provision, deferred_tax_provision.
- `tax_automation_rules`: id (UUID), source_gl_account (JSON), destination_tax_account, logic (NET_OF_TAX/FULL_VALUE).
- `cbcr_reports`: id (UUID), period_id, jurisdiction_id, revenue_related (DECIMAL), profit_loss_before_tax, income_tax_paid, income_tax_accrued.
- `pillar_two_calculations`: id (UUID), period_id, entity_id, globe_income, covered_taxes, top_up_tax_pct, top_up_tax_amount.
- `deferred_tax_assets`: id (UUID), entity_id, period_id, temporary_difference_type, amount, expiry_date.
- `tax_workflows`: id (UUID), provision_id, user_id, action (SUBMIT/REVIEW/APPROVE), timestamp.

## API Endpoints (Axum/gRPC)

- `POST /tax/provisions`: Create a new tax provision for a legal entity.
- `POST /tax/calculate-pillar-two`: Trigger the OECD Pillar Two top-up tax engine.
- `GET /tax/cbcr-report`: Generate the Country-by-Country report for a specific period.
- `GET /tax/provision-vs-actual`: Compare tax provisions with actual tax returns.
- `POST /tax/deferred-assets/track`: Record new temporary differences for deferred tax.
- `GET /tax/dashboard`: View effective tax rates (ETR) across all jurisdictions.

## Inter-service Interaction

- Queries `Financials Service` for pre-tax income and sub-ledger data.
- Queries `EPM Service` (Consolidation) for consolidated group data required for CbCR.
- Emits `Tax_Provision_Finalized` event to `Financials` to create year-end tax journal entries.
- Queries `Tax Service` for real-time statutory tax rates per region.

## Key Logic

- **OECD Pillar Two Compliance:** Calculating Global Anti-Base Erosion (GloBE) income and applying the 15% minimum tax rule across jurisdictions.
- **Tax Automation:** Mapping General Ledger accounts directly to tax line items to reduce manual data entry.
- **Valuation Allowance:** Identifying and flagging deferred tax assets that are unlikely to be realized.
- **Permanent vs. Temporary Differences:** Automatically calculating the tax impact of items like meals & entertainment (permanent) vs. accelerated depreciation (temporary).
- **Consolidated ETR:** Calculating the effective tax rate for the entire corporate group across all entities and countries.
