# Zamazon — PRD Overview

## What Is Zamazon

An Amazon-like e-commerce platform built on a microservices architecture. Supports buyers browsing and purchasing products, sellers listing and managing inventory, and admins operating the platform. Every service is independently deployable and uses the technology best suited to its workload.

---

## Frontends (3 apps)

| App             | Tech       | Port | Users   | Rendering                                                   |
| --------------- | ---------- | ---- | ------- | ----------------------------------------------------------- |
| customer-web    | Next.js 16 | 3000 | Buyers  | SSR on product/category pages, CSR on cart/checkout/account |
| seller-portal   | Next.js 16 | 3001 | Sellers | Mostly CSR, App Router for dashboard layouts                |
| admin-dashboard | Next.js 16 | 3002 | Admins  | Mostly CSR, role-scoped route visibility                    |

All frontends communicate exclusively through the **api-gateway** (port 8000). No frontend calls a backend service directly.

---

## Services (27 backend services)

### P1 — Core Platform _(build first)_

| #   | Service              | Tech   | Port | DB            |
| --- | -------------------- | ------ | ---- | ------------- |
| 1   | api-gateway          | Go     | 8000 | —             |
| 2   | auth-service         | NestJS | 8001 | PostgreSQL    |
| 3   | user-service         | NestJS | 8002 | PostgreSQL    |
| 4   | product-service      | NestJS | 8003 | MongoDB       |
| 5   | inventory-service    | Go     | 8101 | PostgreSQL    |
| 6   | search-service       | Go     | 8105 | Elasticsearch |
| 7   | media-service        | Go     | 8106 | MinIO         |
| 8   | cart-service         | NestJS | 8004 | Redis         |
| 9   | order-service        | Go     | 8102 | PostgreSQL    |
| 10  | payment-service      | Go     | 8103 | PostgreSQL    |
| 11  | wallet-service       | Go     | 8104 | PostgreSQL    |
| 12  | notification-service | NestJS | 8005 | PostgreSQL    |

### P2 — Enhanced Buyer Experience

| #   | Service                | Tech   | Port | DB         |
| --- | ---------------------- | ------ | ---- | ---------- |
| 13  | review-service         | NestJS | 8006 | PostgreSQL |
| 14  | recommendation-service | Django | 8201 | PostgreSQL |
| 15  | deals-service          | NestJS | 8007 | PostgreSQL |
| 16  | promo-service          | NestJS | 8008 | PostgreSQL |
| 17  | loyalty-service        | NestJS | 8009 | PostgreSQL |
| 18  | messaging-service      | NestJS | 8010 | PostgreSQL |
| 19  | support-service        | NestJS | 8011 | PostgreSQL |

### P3 — Seller & Monetization

| #   | Service          | Tech   | Port | DB         |
| --- | ---------------- | ------ | ---- | ---------- |
| 20  | seller-service   | Django | 8203 | PostgreSQL |
| 21  | ads-service      | NestJS | 8012 | PostgreSQL |
| 22  | referral-service | NestJS | 8013 | PostgreSQL |

### P4 — Platform Operations

| #   | Service           | Tech   | Port | DB         |
| --- | ----------------- | ------ | ---- | ---------- |
| 23  | analytics-service | Django | 8202 | PostgreSQL |
| 24  | admin-bff         | Django | 8204 | PostgreSQL |
| 25  | dispute-service   | NestJS | 8014 | PostgreSQL |
| 26  | fraud-service     | Go     | 8107 | PostgreSQL |
| 27  | audit-log-service | Go     | 8108 | PostgreSQL |

---

## System Architecture — Microservices Overview

```
                        ┌─────────────────────────────────┐
  Frontends             │          Cloudflare CDN          │
  ─────────             └─────────────┬───────────────────┘
  customer-web :3000         │
  seller-portal :3001        ▼
  admin-dashboard :3002 ┌──────────────────┐
        │               │   api-gateway    │  Go · :8000
        └───────────────►  Rate limit      │  JWT validation
                        │  Route           │  TLS termination
                        └────────┬─────────┘
                                 │ REST / gRPC
          ┌──────────────────────┼───────────────────────┐
          │                      │                       │
    ┌─────▼──────┐        ┌──────▼─────┐        ┌───────▼──────┐
    │P1 Services │        │P2 Services │        │P3/P4 Services│
    │ auth       │        │ review     │        │ seller       │
    │ user       │        │ recommend  │        │ ads          │
    │ product    │        │ deals      │        │ analytics    │
    │ inventory  │        │ promo      │        │ admin-bff    │
    │ search     │        │ loyalty    │        │ dispute      │
    │ media      │        │ messaging  │        │ fraud        │
    │ cart       │        │ support    │        │ audit-log    │
    │ order      │        └──────┬─────┘        └───────┬──────┘
    │ payment    │               │                      │
    │ wallet     │               │                      │
    │ notify     │               │                      │
    └──────┬─────┘               │                      │
           │                     │                      │
           └─────────────────────┴──────────────────────┘
                                 │
                    ┌────────────▼──────────────┐
                    │      Apache Kafka          │
                    │  (async event bus)         │
                    └────────────┬──────────────┘
                                 │
           ┌─────────────────────┼──────────────────────┐
           │                     │                      │
     ┌─────▼──────┐       ┌──────▼──────┐      ┌───────▼──────┐
     │ PostgreSQL │       │   MongoDB   │      │Elasticsearch │
     │ (primary)  │       │  (catalog)  │      │  (search)    │
     └────────────┘       └─────────────┘      └──────────────┘
           │
     ┌─────▼──────┐       ┌─────────────┐
     │   Redis    │       │    MinIO    │
     │  (cache)   │       │  (storage)  │
     └────────────┘       └─────────────┘
```

### Communication rules

- **Client → api-gateway:** REST over HTTPS
- **Service → Service (sync):** gRPC (internal network only)
- **Service → Service (async):** Kafka topics (fire-and-forget, event-driven)
- **No service exposes itself publicly** — all external traffic goes through api-gateway

---

## Seller KYC Flow

Seller identity verification is managed end-to-end by seller-service — no separate service is needed.

1. The seller uploads their official documents (national ID, business registration, bank statement) through the seller portal
2. The submission enters an admin review queue visible in the admin dashboard
3. An operations admin reviews the documents and approves or rejects the application with a written reason
4. The seller's verification status is updated across the platform — unverified sellers are blocked from listing products and receiving payouts at every point of access
5. The seller is emailed the outcome — approved sellers gain full access immediately; rejected sellers receive the reason and can resubmit

---

## Frontend Architecture Detail

### customer-web (Next.js 16, :3000)

- SSR / Server Components: `/`, `/products/[id]`, `/category/[slug]`, `/search` — SEO-critical
- CSR: `/cart`, `/checkout`, `/account`, `/orders/[id]`
- WebSocket: notification bell, buyer-seller chat (messaging-service)
- State: React Query (server state) + Zustand (client UI state)
- Auth: JWT in HttpOnly cookie

### seller-portal (Next.js 16, :3001)

- Mostly CSR — no SEO needed; App Router nested layouts for sidebar
- Sections: Dashboard, Products, Inventory, Orders, Analytics, Payouts, Ads, Messages
- WebSocket: new order alerts, low-stock notifications
- Auth: JWT in HttpOnly cookie — `seller` role required
- State: React Query + Zustand

### admin-dashboard (Next.js 16, :3002)

- Mostly CSR — internal tool; App Router nested layouts for role-scoped sections
- Auth: JWT in HttpOnly cookie — `admin` role required
- Route visibility controlled by **permission set** in JWT — not just role name
- Connects to admin-bff (:8204) which aggregates data from multiple services
- State: React Query

### Admin Roles & Permission Model

| Role            | Key Permissions                                      | Visible Sections                                    |
| --------------- | ---------------------------------------------------- | --------------------------------------------------- |
| `super_admin`   | `*` all                                              | Everything                                          |
| `operations`    | `users:rw`, `orders:rw`, `disputes:rw`, `sellers:rw` | Users, Orders, Disputes, Sellers                    |
| `moderator`     | `products:moderate`, `reviews:moderate`              | Products, Reviews                                   |
| `finance`       | `payments:read`, `payouts:rw`, `analytics:financial` | Payouts, Financial Reports                          |
| `support_agent` | `tickets:rw`, `users:read`, `orders:read`            | Support Tickets, limited User lookup                |
| `monitor`       | `analytics:read`, `monitoring:read`                  | Analytics dashboards, System Health — **read-only** |

- JWT carries both `role` and `permissions[]` — frontend reads permissions to show/hide routes
- `api-gateway` validates permissions on every request (security layer)
- `admin-bff` validates again before calling downstream services (defense in depth)
- Frontend hiding is UX only — backend always enforces

---

## Infrastructure Summary

| Concern           | Tool                                |
| ----------------- | ----------------------------------- |
| Message broker    | Apache Kafka                        |
| Cache             | Redis                               |
| Relational DB     | PostgreSQL                          |
| Document DB       | MongoDB (product catalog)           |
| Search index      | Elasticsearch                       |
| Object storage    | MinIO (S3-compatible)               |
| CDN               | Cloudflare                          |
| Containers (dev)  | Docker + Docker Compose             |
| Containers (prod) | Kubernetes                          |
| Monitoring        | Prometheus + Grafana                |
| Tracing           | OpenTelemetry + Jaeger              |
| Repo structure    | Monorepo (all services + frontends) |
