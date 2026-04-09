# Fusion AI Data Platform Service Specification

## Overview

The Fusion AI Data Platform service provides a high-performance, unified data layer that merges structured ERP data with unstructured behavioral and external data. It enables real-time AI modeling, vector search, and cross-module analytical insights. This aligns with the Oracle Fusion AI Data Platform (2026 roadmap).

## Data Model (SQLite/Vector Proxy)

- `unified_data_entities`: id (UUID), service_name, entity_name, schema_json (JSON), update_frequency.
- `vector_embeddings_store`: id (UUID), entity_id, record_id, embedding_vector (BLOB), model_version.
- `data_lake_integrations`: id (UUID), source_system (OCI/AWS/AZURE), connection_details (JSON), status.
- `ai_model_registry`: id (UUID), name, type (PREDICTIVE/GENATIVE/CLASSIFICATION), status, training_status.
- `data_lineage_metadata`: id (UUID), record_id, source_service, transformation_steps (JSON), timestamp.
- `real_time_data_streams`: id (UUID), name, event_pattern (JSON), subscriber_count.

## API Endpoints (Axum/gRPC)

- `POST /ai-data-platform/ingest`: Push structured or unstructured data into the unified data platform.
- `GET /ai-data-platform/search/semantic`: Perform a vector-based search across all services (e.g., "Find all vendors with high risk and late shipments").
- `POST /ai-data-platform/models/train`: Request training of a specialized ML model on a specific dataset.
- `GET /ai-data-platform/insights/cross-module`: Retrieve AI-generated correlations (e.g., "Employee sentiment vs. Project margin").
- `POST /ai-data-platform/stream/subscribe`: Subscribe to a real-time stream of unified business events.
- `GET /ai-data-platform/governance/lineage`: View the full history and source of a data point used in an AI prediction.

## Inter-service Interaction

- Queries **ALL Services** to build the unified data lake and vector store.
- Emits `New_Insight_Available` events to the `AI Agent Studio` and `Analytics Service`.
- Receives `Behavioral_Event` from `Behavioral Intelligence` for real-time customer modeling.
- Queries `Identity Service` for data masking and privacy (GDPR) enforcement.
- Emits `Data_Drift_Detected` alert to data scientists.

## Key Logic

- **Vectorization:** Converting business records (Invoices, Resumes, Product Specs) into mathematical vectors for semantic search.
- **Cross-Pillar Correlation:** Identifying hidden relationships between different domains (e.g., "How does factory OEE impact customer satisfaction in B2C Service?").
- **Zero-ETL Integration:** Providing direct access to live service data without complex extraction and loading processes.
- **AI-Driven Data Cleansing:** Automatically identifying and merged duplicate customer or product records using ML.
- **Privacy by Design:** Automatically masking sensitive fields (PII/Financials) based on user roles and regulatory context.
- **Predictive Foundation:** Providing the "Ground Truth" data needed for all other AI agents in the system to operate accurately.
