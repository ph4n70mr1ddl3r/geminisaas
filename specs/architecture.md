# System Architecture Specification

## Technology Stack

- **Backend Framework:** [Axum](https://github.com/tokio-rs/axum) for HTTP/REST and [Tonic](https://github.com/hyperium/tonic) for gRPC.
- **Async Runtime:** `tokio`.
- **Database Access:** `sqlx` or `SeaORM` for SQLite interaction.
- **Serialization:** `serde` for JSON/Protobuf.
- **Authentication:** JWT-based, managed by a central `Identity Service`.
- **Inter-service Communication:**
    - Synchronous: gRPC.
    - Asynchronous: Simple message queue or shared SQLite event bus (depending on scale).
- **Observability:** `tracing` for logging and telemetry.

## Communication Patterns

1.  **API Gateway:** A single entry point (built with Axum) that routes requests to internal microservices.
2.  **Service Mesh (Optional):** If required, though for a clone, direct gRPC calls are preferred for simplicity initially.
3.  **Event-Driven (Saga Pattern):** For distributed transactions (e.g., creating an invoice that updates general ledger), use a saga pattern with compensating transactions.

## Common Components (in `common/`)

- **Error Types:** Unified error handling with `thiserror` and `anyhow`.
- **Data Models:** Shared DTOs for common entities like `Money`, `Date`, `Address`.
- **Auth Middleware:** Shared middleware for JWT validation.

## Security

- All service-to-service communication must be authenticated.
- Role-Based Access Control (RBAC) is enforced at the service level.
- Data at rest (SQLite files) should be encrypted if sensitive.
