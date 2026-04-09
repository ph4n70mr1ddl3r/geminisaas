# Digital Assistant (AI Agent Studio) Service Specification

## Overview

The Digital Assistant (AI Agent Studio) service provides a conversational AI platform and a unified workspace for deploying role-specific AI agents across all ERP, SCM, HCM, and CX pillars. This aligns with Oracle Digital Assistant (ODA).

## Data Model (SQLite)

- `ai_agents`: id (UUID), name, role (EXPENSES/SALES/PLANNING/HR), status (ACTIVE/DRAFT), capabilities (JSON).
- `ai_skills`: id (UUID), agent_id, intent_name, description, training_data (JSON), status.
- `ai_conversations`: id (UUID), user_id, status (ACTIVE/COMPLETED), last_message_time, transcript (JSON).
- `ai_agent_permissions`: id (UUID), agent_id, role_id, access_level (READ/WRITE/EXECUTE).
- `ai_agent_configs`: id (UUID), agent_id, theme_color, welcome_message, fallback_message.
- `ai_agent_metrics`: id (UUID), agent_id, user_id, intent_matched_pct, success_pct, timestamp.

## API Endpoints (Axum/gRPC)

- `POST /digital-assistant/message`: Send a message to the conversational AI and receive an agent-led response.
- `GET /digital-assistant/agents`: Retrieve a list of available AI agents for the current user.
- `POST /digital-assistant/skills/deploy`: Deploy a new intent or skill to an agent.
- `GET /digital-assistant/history/{user_id}`: Retrieve past conversation transcripts for a specific user.
- `POST /digital-assistant/agent/configure`: Update look-and-feel and welcome messages for an agent.
- `GET /digital-assistant/analytics/intent-usage`: Retrieve metrics on which intents are most frequently used.

## Inter-service Interaction

- Queries **ALL Services** for domain-specific data to provide conversational answers (e.g., querying `Financials` for invoice status).
- Emits `Action_Triggered_By_AI` event to relevant services to perform a task (e.g., submitting an expense in `Expenses Service`).
- Queries `Identity Service` for user roles to ensure the AI only provides data the user is authorized to see.
- Emits `AI_Feedback_Received` to the agent training loop for continuous improvement.

## Key Logic

- **Natural Language Understanding (NLU):** Using LLMs to understand user intent, extract entities, and handle complex multi-turn conversations.
- **Agent Orchestration:** Automatically routing a user's request to the correct specialized agent (e.g., routing an "I'm sick" message to the HR agent).
- **Proactive Notifications:** Initiating a conversation with a user when a critical event occurs (e.g., "Your expense report was rejected, would you like to know why?").
- **Agentic Workflows:** Allowing an agent to combine multiple steps across services to complete a goal (e.g., identifying a stockout in `WMS` and creating a transfer order in `Inventory`).
- **Context Awareness:** Remembering the context of a conversation (e.g., knowing that "it" refers to the invoice mentioned in the previous message).
- **Redwood Visual Integration:** Presenting data-rich UI cards (e.g., graphs, tables) within the chat interface for better visualization.
