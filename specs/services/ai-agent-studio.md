# AI Agent Studio Service Specification

## Overview

The AI Agent Studio service provides a low-code environment for organizations to configure, train, and deploy autonomous AI agents across the Fusion Cloud ecosystem. These agents go beyond simple assistance to perform multi-step tasks, monitor for exceptions, and execute business logic within defined guardrails. This aligns with the Oracle AI Agent Studio roadmap for 2026-2027.

## Data Model (SQLite)

- `ai_agent_definitions`: id (UUID), name, base_role (FINANCE/SCM/HR/SALES), status (ACTIVE/DRAFT), security_context_id.
- `agent_knowledge_bases`: id (UUID), agent_id, source_type (SERVICE_API/DOC_LINK/GL_QUERY), status.
- `agent_action_skills`: id (UUID), agent_id, skill_name, executable_code_ref (Wasm/Python), approval_required (BOOL).
- `agent_guardrails`: id (UUID), agent_id, constraint_type (SPENDING_LIMIT/DATA_PRIVACY/ACCESS_SCOPE), limit_value.
- `agent_execution_logs`: id (UUID), agent_id, user_request, action_taken, outcome (SUCCESS/FAILED), feedback_score.
- `agent_orchestration_chains`: id (UUID), name, steps_json (JSON - sequence of multiple agents working together).

## API Endpoints (Axum/gRPC)

- `POST /ai-agent-studio/agents`: Define a new autonomous agent with specific goals and constraints.
- `POST /ai-agent-studio/train`: Provide local context or documents to enhance an agent's domain knowledge.
- `GET /ai-agent-studio/catalog`: Retrieve a list of pre-built and custom agents available for deployment.
- `POST /ai-agent-studio/execute`: Trigger an agent to perform a complex task (e.g., "Resolve all pending receipt accruals older than 90 days").
- `GET /ai-agent-studio/analytics/performance`: View agent success rates, time saved, and user feedback.
- `POST /ai-agent-studio/guardrails/update`: Modify the safety limits for an active agent.

## Inter-service Interaction

- Queries **ALL Services** to retrieve data and execute actions via `agent_action_skills`.
- Queries `Identity Service` to ensure the agent operates with the same permissions as the initiating user.
- Emits `Agent_Action_Performed` event for audit and compliance in `Advanced Usage Controls (AUC)`.
- Receives `System_Exception` events to trigger proactive agent remediation.
- Queries `Enterprise Data Management (EDM)` for master data hierarchies used in decision making.

## Key Logic

- **Autonomous Task Execution:** Decomposing a high-level goal (e.g., "Minimize shipping delays") into a series of API calls across SCM services.
- **Dynamic Context Injection:** Automatically retrieving relevant historical data to provide "few-shot" examples to the underlying LLM.
- **Human-in-the-Loop:** Automatically pausing and requesting approval if an agent's proposed action exceeds a safety threshold.
- **Agent Collaboration:** Enabling a "Sourcing Agent" to talk to a "Logistics Agent" to find the most cost-effective way to expedite a late order.
- **Continuous Learning:** Refining agent behavior based on user corrections and successful outcomes.
- **Contextual UI Embedding:** Displaying agent suggestions and "AI Assist" buttons directly within standard Redwood pages.
