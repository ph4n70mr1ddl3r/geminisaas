# Database Strategy Specification

## SQLite in Microservices

Each microservice manages its own SQLite database file. This ensures isolation and scalability.

### Configuration

- **Journal Mode:** `WAL` (Write-Ahead Logging) for better concurrency.
- **Synchronous Mode:** `NORMAL` (safe for WAL, faster performance).
- **Busy Timeout:** `5000ms` to prevent immediate "database locked" errors.
- **Foreign Keys:** Enabled by default (`PRAGMA foreign_keys = ON;`).

### Migrations

- Use `sqlx-cli` or `SeaORM` migrations to manage schema changes.
- Each service contains its own `migrations/` folder.
- Migrations must be run automatically on service startup or via CI/CD.

### Data Modeling

- Use UUIDs (v7 for time-sorting) as primary keys.
- Store monetary values as `INTEGER` (cents/minor units) or using a dedicated `Decimal` type handled by the application logic.
- Timestamps: Always in UTC, stored as ISO-8601 strings or integers.

### Backups & Replication

- **Litestream:** Recommended for continuous backup to S3-compatible storage.
- **LiteFS:** Consider for read-replication if horizontal scaling becomes necessary.
- Point-in-time recovery must be configured for production environments.

### Shared Data

- Services should *never* share a database file.
- Master data (e.g., Currency codes, Country codes) should be synchronized via an `Internal Data Service` or duplicated and kept in sync via events.
