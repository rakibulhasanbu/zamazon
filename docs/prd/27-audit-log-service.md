# Audit Log Service

**Technology:** Go
**Port:** 8108
**Priority:** P4 — Platform Operations

---

## Why Go

The audit log consumes every significant event from every service on the platform — this is the highest-volume write workload in the entire system. Go's lightweight goroutines handle thousands of concurrent Kafka message consumptions with minimal memory overhead. There are no GC pauses that would cause log entries to be dropped during traffic spikes. The service itself has almost no business logic — it receives events and writes immutable records — and Go is the most efficient choice for pure high-throughput I/O.

---

## What This Service Does

- Automatically records every significant action that happens anywhere on the platform — order placed, payment processed, product listed, account suspended, dispute resolved, admin permission changed — without any service needing to call this service directly; it consumes Kafka events passively
- Every log record is immutable — once written, it cannot be edited or deleted through normal application paths, providing a tamper-resistant history
- Stores the full context of each event: who performed the action, what resource was affected, when it happened, what the outcome was, the source IP address, and the requesting admin's identity for admin actions
- Makes logs searchable by admins with `audit:read` permission through the admin dashboard — they can look up all actions taken by a specific user, all actions on a specific order, or all events within a time range
- Implements hash chaining between log entries — each record includes a hash of the previous record, so any tampering with historical entries is mathematically detectable
- Enforces a retention policy — logs older than the defined retention period are automatically archived to cold storage rather than deleted, ensuring they remain available for compliance purposes
- Supports compliance exports — an admin can generate a downloadable report of all audit events for a specific user or time period, for use in legal, compliance, or regulatory processes

---

## Who Uses It

- **Admins (super_admin only for full access, audit:read permission required)** — search and review the audit trail through the admin dashboard
- **System** — every other service feeds events into this one indirectly via Kafka; no service calls it directly

---

## Key Integrations

- Consumes every significant Kafka topic across the platform: `order.*`, `payment.*`, `wallet.*`, `user.registered`, `dispute.*`, `fraud.flagged`, `support.ticket.*`, and all others
- Provides searchable audit data to admin-bff for the audit log viewer in the admin dashboard
- Archives old records to cold storage (MinIO) on a scheduled basis
