# Grants Management Service Specification

## Overview

The Grants Management service handles the lifecycle of externally funded projects, including awards, funding sources, and specific compliance requirements. This aligns with Oracle Fusion Cloud Grants Management.

## Data Model (SQLite)

- `awards`: id (UUID), name, number, sponsor_id (link to Vendor/Customer), start_date, end_date, amount, status (ACTIVE/CLOSED/PRE_AWARD).
- `funding_sources`: id (UUID), award_id, source_type (FEDERAL/STATE/PRIVATE), amount, restriction_rules (JSON).
- `grant_projects`: id (UUID), award_id, project_id (link to Project Management), budget_allocation.
- `grant_costs`: id (UUID), grant_project_id, amount, date, cost_type (DIRECT/INDIRECT), status (APPROVED/REJECTED).
- `indirect_cost_rates`: id (UUID), project_id, rate_pct, base_cost_pool (JSON).
- `compliance_reports`: id (UUID), award_id, type (FINANCIAL/TECHNICAL), due_date, status.

## API Endpoints (Axum/gRPC)

- `POST /grants/awards`: Record a new grant award from a sponsor.
- `GET /grants/awards/{id}/funding`: View funding distribution across projects.
- `POST /grants/projects/link`: Associate a project with a grant award.
- `GET /grants/burn-rate`: Analyze how quickly grant funds are being spent.
- `POST /grants/reports/generate`: Create required financial reporting for a sponsor.

## Inter-service Interaction

- Receives `Project_Cost_Recorded` from `Project Management` to validate against grant rules.
- Queries `Financials Service` for payment and reconciliation status from sponsors.
- Emits `Grant_Budget_Exceeded` to `Procurement` and `HCM` to stop spending.
- Queries `Identity Service` for Principal Investigator (PI) and administrative approvals.

## Key Logic

- **Direct vs. Indirect Costs:** Automatically calculating and applying "overhead" (indirect) costs based on the direct spend (e.g., 20% on top of labor).
- **Cost Sharing:** Managing cases where the organization must contribute a portion of the project cost (matching funds).
- **Effort Certification:** Logic for ensuring that labor charged to a grant matches the actual effort reported by researchers.
- **Sponsor Constraints:** Enforcing specific rules for what can be purchased with grant funds (e.g., "no international travel").
- **Award Closeout:** Automating the final financial reconciliation when a grant expires.
