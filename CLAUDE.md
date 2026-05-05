# Zamazon — Project Context

## What This Is

An Amazon-like e-commerce platform built as a portfolio project. Full microservices architecture with 27 backend services, 3 Next.js frontends, and a proper build roadmap (idea → PRD → SRS → architecture → implementation).

**Two frameworks only: Go and NestJS.** No Django.
- **Go** — high-throughput, latency-sensitive, or compute-heavy services
- **NestJS** — complex business logic, workflows, state machines, validation-heavy services

---

## Build Roadmap & Current State

| Phase | Artifact | Status |
|---|---|---|
| 0 | `idea.md` | Done |
| 1 | `docs/prd/` | Done (28 files) |
| 2 | `docs/srs/` | Next |
| 3 | `docs/architecture/` | Not started |
| 4 | `services/*/docs/` | Not started |
| 5 | Service implementation | Not started |
| 6 | Inter-service integration | Not started |
| 7 | DevOps (Docker / K8s) | Not started |
| 8 | Testing | Not started |

---

## All 27 Services

### P1 — Core Platform (build first)

| # | Service | Tech | Port | DB |
|---|---|---|---|---|
| 1 | api-gateway | Go | 8000 | — |
| 2 | auth-service | NestJS | 8001 | PostgreSQL |
| 3 | user-service | NestJS | 8002 | PostgreSQL |
| 4 | product-service | NestJS | 8003 | MongoDB |
| 5 | inventory-service | Go | 8101 | PostgreSQL |
| 6 | search-service | Go | 8105 | Elasticsearch |
| 7 | media-service | Go | 8106 | MinIO |
| 8 | cart-service | NestJS | 8004 | Redis |
| 9 | order-service | Go | 8102 | PostgreSQL |
| 10 | payment-service | Go | 8103 | PostgreSQL |
| 11 | wallet-service | Go | 8104 | PostgreSQL |
| 12 | notification-service | NestJS | 8005 | PostgreSQL |

### P2 — Enhanced Buyer Experience

| # | Service | Tech | Port | DB |
|---|---|---|---|---|
| 13 | review-service | NestJS | 8006 | PostgreSQL |
| 14 | recommendation-service | NestJS | 8201 | PostgreSQL |
| 15 | deals-service | NestJS | 8007 | PostgreSQL |
| 16 | promo-service | NestJS | 8008 | PostgreSQL |
| 17 | loyalty-service | NestJS | 8009 | PostgreSQL |
| 18 | messaging-service | NestJS | 8010 | PostgreSQL |
| 19 | support-service | NestJS | 8011 | PostgreSQL |

### P3 — Seller & Monetization

| # | Service | Tech | Port | DB |
|---|---|---|---|---|
| 20 | seller-service | NestJS | 8203 | PostgreSQL |
| 21 | ads-service | NestJS | 8012 | PostgreSQL |
| 22 | referral-service | NestJS | 8013 | PostgreSQL |

### P4 — Platform Operations

| # | Service | Tech | Port | DB |
|---|---|---|---|---|
| 23 | analytics-service | Go | 8202 | PostgreSQL |
| 24 | admin-bff | NestJS | 8204 | PostgreSQL |
| 25 | dispute-service | NestJS | 8014 | PostgreSQL |
| 26 | fraud-service | Go | 8107 | PostgreSQL |
| 27 | audit-log-service | Go | 8108 | PostgreSQL |

---

## Frontends (3 Next.js 16 apps)

| App | Port | Users | Notes |
|---|---|---|---|
| customer-web | 3000 | Buyers | SSR on product/category (SEO), CSR on cart/checkout |
| seller-portal | 3001 | Sellers | Mostly CSR, App Router dashboard layouts |
| admin-dashboard | 3002 | Admins | CSR, role-scoped route visibility |

All frontends talk exclusively to **api-gateway** (:8000). No direct service calls from any frontend.

---

## Infrastructure

| Concern | Tool |
|---|---|
| Message broker | Apache Kafka |
| Cache | Redis |
| Relational DB | PostgreSQL |
| Document DB | MongoDB (product catalog only) |
| Search | Elasticsearch |
| Object storage | MinIO (S3-compatible) |
| CDN | Cloudflare |
| Containers (dev) | Docker + Docker Compose |
| Containers (prod) | Kubernetes |
| Monitoring | Prometheus + Grafana |
| Tracing | OpenTelemetry + Jaeger |

---

## Communication

- **Client → api-gateway:** REST over HTTPS
- **Service → Service (sync):** gRPC
- **Service → Service (async):** Kafka topics
- No service is publicly exposed — all external traffic goes through api-gateway

---

## Key Design Decisions

- **payment-service vs wallet-service** — payment talks to external gateways (Stripe, PayPal); wallet is an internal ledger. A Stripe outage never blocks wallet reads.
- **messaging-service vs support-service** — real-time P2P chat (WebSocket) vs async LLM ticket Q&A (1–5s latency). Mixing them would couple chat to AI inference time.
- **inventory-service is separate** — stock must be atomically reserved under concurrent checkout writes; critical path.
- **affiliate = program membership, not a role** — auth-service has 3 roles: `buyer`, `seller`, `admin`. Affiliate status lives in referral-service as a record linked to `user_id`.
- **admin-bff owns no state** — it aggregates and routes. KYC state lives in seller-service; dispute state in dispute-service, etc.

## Admin Roles

| Role | Permissions |
|---|---|
| `super_admin` | everything |
| `operations` | users, sellers, orders, disputes |
| `moderator` | products, reviews (moderation only) |
| `finance` | payments, payouts, financial reports |
| `support_agent` | tickets, limited user/order lookup |
| `monitor` | read-only dashboards and system health |

JWT carries both `role` and `permissions[]`. api-gateway validates on every request; admin-bff validates again (defense in depth).

---

## Where to Find Things

| What | Path |
|---|---|
| Full vision & design decisions | `idea.md` |
| PRD overview (service table, arch diagram) | `docs/prd/overview.md` |
| Per-service PRD | `docs/prd/NN-<service-name>.md` |
| SRS (not yet written) | `docs/srs/` |
| Architecture docs (not yet written) | `docs/architecture/` |
