# B2C Service Specification

## Overview

The B2C Service (formerly Oracle RightNow) manages high-volume customer interactions across digital channels, including chat, email, social media, and self-service. This aligns with Oracle Fusion Cloud B2C Service.

## Data Model (SQLite)

- `incidents`: id (UUID), customer_id (link to CDM), subject, description, priority, status (UNASSIGNED/ASSIGNED/SOLVED/CLOSED), channel (WEB/CHAT/SOCIAL), assigned_agent_id.
- `incident_threads`: id (UUID), incident_id, sender_id, text, type (CUSTOMER/AGENT/SYSTEM), timestamp.
- `service_chat_sessions`: id (UUID), incident_id, status (ACTIVE/COMPLETED), duration_min, transcript.
- `customer_assets`: id (UUID), customer_id, product_id, serial_number, installation_date, status.
- `agent_availability`: id (UUID), agent_id, status (AVAILABLE/BUSY/OFFLINE), skills (JSON).
- `service_routing_rules`: id (UUID), criteria (JSON), target_queue_id, priority.

## API Endpoints (Axum/gRPC)

- `POST /b2c-service/incidents`: Create a new customer service request.
- `GET /b2c-service/incidents/search`: Search for incidents based on status or customer.
- `POST /b2c-service/chat/start`: Begin a real-time digital chat session.
- `POST /b2c-service/incidents/{id}/reply`: Send a message to the customer from an agent.
- `GET /b2c-service/agent/inbox`: Retrieve pending tasks for the current agent.
- `POST /b2c-service/incidents/assign`: Route an incident to the best-fit agent or queue.

## Inter-service Interaction

- Queries `CDM Service` for a 360-degree view of the customer and their past history.
- Queries `Knowledge Management` to suggest relevant articles to the agent or customer.
- Emits `Incident_Solved` event to `Marketing` for follow-up satisfaction surveys.
- Queries `Intelligent Advisor` for automated, rule-based troubleshooting interviews.
- Emits `Part_Required` event to `Service Logistics` if a field repair is needed.

## Key Logic

- **Automated Routing:** Using skills-based and workload-based logic to assign an incident to the right agent.
- **Sentiment Analysis:** Automatically flagging incidents with high frustration levels for urgent attention.
- **Thread Summarization:** Using AI to create a concise summary of a long customer interaction for quick agent review.
- **SLA Management:** Tracking and alerting when an incident is nearing its response or resolution time limit.
- **Channel Switching:** Allowing a customer to start an interaction on social media and seamlessly move to email or chat without losing context.
- **Standard Text Suggestions:** Recommending pre-written, approved responses to agents based on the incident's subject.
