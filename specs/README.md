# Oracle Fusion Cloud ERP Clone - Spec-Driven Development

This repository contains the specifications for building a high-performance, microservices-based ERP system, designed to be a functional clone of Oracle Fusion Cloud ERP.

## Core Principles

1.  **Spec-Driven Development:** Every feature must be defined in a `.md` specification before implementation. AI agents use these specs as the source of truth.
2.  **Full-Stack Rust:** Both the frontend (Leptos/Dioxus/Yew) and the backend (Axum/Actix/Poise) are built using Rust for performance and reliability.
3.  **Microservices Architecture:** Each domain (Financials, Procurement, etc.) is its own service, communicating via gRPC or message queues.
4.  **SQLite as Database:** Each microservice manages its own SQLite database, using WAL mode for concurrency and litestream/LiteFS for replication/backups.
5.  **AI-First Implementation:** All code is generated or assisted by AI agents following the definitions in `specs/`.

## Directory Structure

- `specs/`: Root for all specification documents.
    - `architecture.md`: System-wide architectural standards, common patterns, and communication protocols.
    - `database.md`: SQLite configuration, migration strategies, and data modeling patterns.
    - `frontend.md`: UI/UX standards, design system, and frontend-backend interaction.
    - `services/`: Specific functional modules (e.g., `financials.md`, `procurement.md`).
    - `common/`: Shared definitions (e.g., common data types, error handling, auth).

## Agent Instructions

When implementing a service:
1.  Read the relevant module spec in `specs/services/`.
2.  Consult `specs/architecture.md` for patterns and standards.
3.  Consult `specs/database.md` for SQLite setup and ORM conventions.
4.  Implement the service in its own crate under `services/`.
5.  Validate against the spec before marking as complete.
