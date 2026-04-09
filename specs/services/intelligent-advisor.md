# Intelligent Advisor Service Specification

## Overview

The Intelligent Advisor (formerly Oracle Policy Automation) service provides automated decision-making and personalized customer interviews based on complex business rules and regulations. This aligns with Oracle Fusion Cloud Intelligent Advisor.

## Data Model (SQLite)

- `ia_deployments`: id (UUID), name, status (ACTIVE/ARCHIVED), version, last_deployed.
- `ia_rulesets`: id (UUID), deployment_id, source_language (NATURAL_LANGUAGE), rules_logic (JSON), version.
- `ia_interviews`: id (UUID), deployment_id, user_id, status (IN_PROGRESS/COMPLETED), data_snapshot (JSON).
- `ia_attributes`: id (UUID), ruleset_id, name, type (BOOLEAN/NUMBER/STRING/DATE), mapped_entity_id.
- `ia_outcomes`: id (UUID), interview_id, decision_made, explanation (Markdown), confidence_score.
- `ia_mapping_configs`: id (UUID), attribute_id, external_service, external_field.

## API Endpoints (Axum/gRPC)

- `POST /ia/deploy`: Publish a new ruleset from a Natural Language source (Word/Excel).
- `POST /ia/interview/start`: Begin a personalized data collection session.
- `POST /ia/interview/next-step`: Submit current answers and receive the next set of questions or a decision.
- `GET /ia/explain/{interview_id}`: Retrieve a detailed audit trail of why a decision was reached.
- `POST /ia/batch-assess`: Run thousands of records against a ruleset in a single operation.
- `GET /ia/analytics/hotspots`: Identify where users most frequently drop out of an interview.

## Inter-service Interaction

- Queries `CRM Service` for customer context to pre-populate interview answers.
- Emits `Decision_Reached` event to `Procurement`, `HCM`, or `CRM` to trigger an automated action.
- Queries `Financials Service` for real-time data values to use in rules (e.g., spending thresholds).
- Emits `Interview_Completed` event to `Marketing` with captured data for personalization.

## Key Logic

- **Natural Language Rules:** Converting business-friendly language (e.g., "If age is > 18 then person is adult") into an executable decision engine.
- **Dynamic Interview Flow:** Only asking questions that are relevant based on previous answers (skip-logic).
- **Explanation Engine:** Providing a human-readable "Decision Report" that cites the specific rules and logic used to arrive at a result.
- **Data Mapping:** Automatically loading and saving data back to the core ERP/CRM records after an interview.
- **Temporal Reasoning:** Handling complex rules that change over time (e.g., "Effective Jan 1, the new regulation applies").
