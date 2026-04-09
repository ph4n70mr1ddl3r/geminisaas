# Innovation Management Service Specification

## Overview

The Innovation Management service provides a platform for capturing, evaluating, and developing new product ideas, proposals, and concepts before they become formal products in the PLM system. This aligns with Oracle Fusion Cloud Innovation Management.

## Data Model (SQLite)

- `product_ideas`: id (UUID), title, description, creator_id, status (SUBMITTED/REVIEW/ACCEPTED/REJECTED), votes_count, category_id.
- `innovation_proposals`: id (UUID), idea_id, name, target_market, estimated_revenue, estimated_cost, status (DRAFT/SUBMITTED/APPROVED).
- `innovation_concepts`: id (UUID), proposal_id, name, technical_specs (JSON), status (DRAFT/PROTOTYPE/VALIDATED).
- `innovation_campaigns`: id (UUID), name, theme, start_date, end_date, total_ideas_submitted.
- `innovation_comments`: id (UUID), item_id, user_id, text, timestamp.
- `innovation_portfolio`: id (UUID), name, projects (JSON), total_value, risk_profile (LOW/MED/HIGH).

## API Endpoints (Axum/gRPC)

- `POST /innovation/ideas`: Submit a new product or improvement idea.
- `POST /innovation/ideas/{id}/vote`: Record user interest in an idea.
- `POST /innovation/proposals`: Create a business case for a promising idea.
- `POST /innovation/concepts`: Define the technical and functional aspects of a proposal.
- `GET /innovation/campaigns`: List active innovation challenges.
- `POST /innovation/promote`: Transition a validated concept into a formal product (PLM).

## Inter-service Interaction

- Queries `PLM Service` to check if similar products already exist.
- Emits `Product_Concept_Approved` event to `PLM` to initiate formal item creation.
- Queries `CRM Service` for market trends and customer feedback to inform ideas.
- Emits `Innovation_Award_Recommended` to `HCM` for employee recognition.
- Queries `EPM Service` (Planning) for long-term R&D budget alignment.

## Key Logic

- **Idea Crowdsourcing:** Allowing employees and partners to submit and vote on ideas to identify the most promising ones.
- **Business Case Modeling:** Providing a structured format for estimating ROI, market size, and strategic fit for a proposal.
- **Concept Prototype Management:** Tracking the evolution of a concept through various stages of technical validation.
- **Campaign Management:** Creating time-bound challenges to drive innovation in specific areas (e.g., "Sustainability").
- **Portfolio Balancing:** Analyzing the mix of innovation projects to ensure a balance of risk and potential reward.
- **Redwood Collaboration:** Using modern UI patterns to facilitate brainstorming and feedback loops.
