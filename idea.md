# Zamazon — Amazon-like E-Commerce Platform

## Vision

A portfolio-grade, production-minded e-commerce platform built on a microservices architecture. Every technology choice is deliberate — NestJS and Go are each used where they perform best, not interchangeably.

**Core user journeys:**
Browse products → Search & filter → Add to cart → Checkout → Pay → Track order

**Out of scope:** logistics network, video/music streaming, AWS marketplace, physical stores.

---

## Build Roadmap

| Phase | Artifact                  | Description                                              |
| ----- | ------------------------- | -------------------------------------------------------- |
| 0     | `idea.md`                 | Vision, tech map, service list (this file)               |
| 1     | `docs/prd/`               | Product Requirements — personas, features, NFRs          |
| 2     | `docs/srs/`               | Software Requirements — use cases, data flows, contracts |
| 3     | `docs/architecture/`      | System design, DB schema, API design, infra              |
| 4     | `services/*/docs/`        | Per-service design docs                                  |
| 5     | Service implementation    | Code, bottom-up: infra → core → edge                     |
| 6     | Inter-service integration | Kafka events, gRPC wiring                                |
| 7     | DevOps                    | Docker Compose (dev) → Kubernetes (prod)                 |
| 8     | Testing                   | Unit → Integration → E2E                                 |

---

## Service Priority & Port Reference

Services are ordered by build priority — P1 must work before P2 can be built, and so on.

| #   | Service                | Tech   | Port | DB            | Priority  |
| --- | ---------------------- | ------ | ---- | ------------- | --------- |
| 1   | api-gateway            | Go     | 8000 | —             | P1 Core   |
| 2   | auth-service           | NestJS | 3001 | PostgreSQL    | P1 Core   |
| 3   | user-service           | NestJS | 3002 | PostgreSQL    | P1 Core   |
| 4   | product-service        | NestJS | 3003 | MongoDB       | P1 Core   |
| 5   | inventory-service      | Go     | 4001 | PostgreSQL    | P1 Core   |
| 6   | search-service         | Go     | 4005 | Elasticsearch | P1 Core   |
| 7   | media-service          | Go     | 4006 | MinIO         | P1 Core   |
| 8   | cart-service           | NestJS | 3004 | Redis         | P1 Core   |
| 9   | order-service          | Go     | 4002 | PostgreSQL    | P1 Core   |
| 10  | payment-service        | Go     | 4003 | PostgreSQL    | P1 Core   |
| 11  | wallet-service         | Go     | 4004 | PostgreSQL    | P1 Core   |
| 12  | notification-service   | NestJS | 3005 | PostgreSQL    | P1 Core   |
| 13  | review-service         | NestJS | 3006 | PostgreSQL    | P2 Buyer  |
| 14  | recommendation-service | NestJS | 5001 | PostgreSQL    | P2 Buyer  |
| 15  | deals-service          | NestJS | 3007 | PostgreSQL    | P2 Buyer  |
| 16  | promo-service          | NestJS | 3008 | PostgreSQL    | P2 Buyer  |
| 17  | loyalty-service        | NestJS | 3009 | PostgreSQL    | P2 Buyer  |
| 18  | messaging-service      | NestJS | 3010 | PostgreSQL    | P2 Buyer  |
| 19  | support-service        | NestJS | 3011 | PostgreSQL    | P2 Buyer  |
| 20  | seller-service         | NestJS | 5003 | PostgreSQL    | P3 Seller |
| 21  | ads-service            | NestJS | 3012 | PostgreSQL    | P3 Seller |
| 22  | referral-service       | NestJS | 3013 | PostgreSQL    | P3 Seller |
| 23  | analytics-service      | Go     | 5002 | PostgreSQL    | P4 Ops    |
| 24  | admin-bff              | NestJS | 5004 | PostgreSQL    | P4 Ops    |
| 25  | dispute-service        | NestJS | 3014 | PostgreSQL    | P4 Ops    |
| 26  | fraud-service          | Go     | 4007 | PostgreSQL    | P4 Ops    |
| 27  | audit-log-service      | Go     | 4008 | PostgreSQL    | P4 Ops    |

---

## Services (27 total)

Each service entry lists: the tech used, why it was chosen, and the exact features it owns.

---

### 1. api-gateway — **Go**

**Why Go:** Needs to be the fastest layer in the stack. Handles every inbound request before it reaches any service — Go's net/http and goroutine model make it ideal for a high-throughput, low-overhead proxy.

**Features covered:**

- Route all client requests to the correct downstream service
- JWT validation at the edge (reject invalid tokens before hitting any service)
- Rate limiting per user / IP
- Request logging and correlation ID injection
- TLS termination

---

### 2. auth-service — **NestJS**

**Why NestJS:** Passport.js, Guards, JWT strategy, and OAuth2 adapters are all first-class citizens. Complex auth flows (refresh tokens, OAuth handshake) benefit from NestJS's decorator-based structure.

**Features covered:**

- User registration (email + password)
- Login / logout
- OAuth2 login (Google, Facebook)
- JWT access token + refresh token flow
- Password reset via email OTP
- Account email verification
- Seller KYC / account verification initiation
- Role-based access control (buyer, seller, admin)

---

### 3. user-service — **NestJS**

**Why NestJS:** Profile management is CRUD-heavy with complex validation rules. TypeORM and class-validator handle this cleanly. Integrates well with auth-service events.

**Features covered:**

- User profile (name, avatar, bio)
- Address book (add, edit, delete, set default)
- Wishlist management
- Recently viewed products (stored per user)
- Account settings & preferences
- Buyer order history (read-only view, sourced from order-service)

---

### 4. product-service — **NestJS**

**Why NestJS:** Product catalog has rich relational data — categories, subcategories, variants, attributes, and seller associations. TypeORM with PostgreSQL handles this schema well.

**Features covered:**

- Product CRUD (create, read, update, delete)
- Category & subcategory management
- Product variants (size, color, material, etc.)
- Product attributes & specifications
- Product images (links to media-service)
- Dynamic pricing rules (base price, sale price, bulk discount)
- Subscribe & Save eligibility flag per product
- Q&A on products (questions from buyers, answers from sellers)
- Product status (draft, active, archived)

---

### 5. inventory-service — **Go**

**Why Go:** Stock operations must be atomic under concurrent writes (two buyers checking out the last unit simultaneously). Goroutines + database-level locking via pgx give precise control.

**Features covered:**

- Stock level tracking per product/variant/warehouse
- Atomic stock reservation on checkout
- Stock release on order cancellation
- Low stock alerts (triggers Kafka event → notification-service)
- Inventory adjustment history (manual corrections)
- Multi-warehouse stock aggregation

---

### 6. cart-service — **NestJS**

**Why NestJS:** Redis-backed session cart is straightforward with the NestJS cache/Redis module. Event-driven sync on price changes fits the NestJS event emitter pattern.

**Features covered:**

- Add / update / remove items in cart
- Guest cart (session-based) → merge on login
- Cart price recalculation (reflects live product prices)
- Apply coupon / promo code to cart
- Cart item count badge sync
- Save cart for later

---

### 7. order-service — **Go**

**Why Go:** Order creation is the most critical high-throughput write path. Saga orchestration for distributed transactions (reserve stock → charge payment → confirm order) needs precise control flow — Go's explicit error handling makes failure paths safe.

**Features covered:**

- Place order (single and multi-seller)
- Order confirmation & reference number generation
- Order status lifecycle (pending → confirmed → shipped → delivered → completed)
- Order cancellation (with stock release event)
- Re-order (clone past order into cart)
- Subscribe & Save recurring order scheduling
- Invoice generation per order
- Flash Deal / Lightning Deal order validation (time-window enforcement)
- Dispute initiation (links to dispute-service)

---

### 8. payment-service — **Go**

**Why Go:** Payment processing must be deterministic, fast, and free of GC pauses. Go's explicit error handling and minimal runtime overhead are the right fit for financial operations.

**Scope boundary:** This service only talks to the external world (payment gateways). Internal balance/earnings/wallet logic lives in wallet-service. The split exists because external gateway failures should not affect internal ledger operations — they fail for different reasons.

**Features covered:**

- Multiple payment gateway integration (Stripe, PayPal, etc.)
- Secure card tokenization (no raw card data stored)
- Payment capture, refund, and partial refund
- Execute bank transfer payouts (triggered by wallet-service withdrawal requests)
- Fraud signal emission (to fraud-service)
- Payment webhook handling (async gateway callbacks)
- Emit `payment.confirmed` / `payment.failed` events to Kafka

---

### 8b. wallet-service — **Go**

**Why Go:** The wallet is a high-frequency internal ledger. Atomic balance updates under concurrent reads/writes (two redemptions at the same moment) need Go's concurrency model. No external API calls — pure internal operations.

**Why separate from payment-service:** wallet-service is a pure internal double-entry ledger with no network dependencies. payment-service calls external gateways. Separating them means a Stripe outage doesn't block wallet reads, and wallet bugs don't touch payment gateway logic.

**Features covered:**

- Internal wallet balance per user (store credit, cashback, referral rewards)
- Seller earnings ledger (credits on order.completed, debits on payout)
- Wallet top-up (via payment-service charge)
- Wallet spend at checkout (partial or full payment)
- Withdrawal request creation (buyer/seller requests payout → payment-service executes)
- Withdrawal status tracking (pending, processing, completed, failed)
- Transaction history (all credits and debits with reason codes)
- Balance lock during checkout (reserve, confirm, or release)

---

### 9. search-service — **Go**

**Why Go:** Elasticsearch query throughput at read-heavy load. Go's HTTP client with connection pooling keeps latency low across thousands of search requests per second.

**Features covered:**

- Full-text product search
- Filters (price range, rating, category, brand, availability)
- Sort (relevance, price, newest, rating)
- Search suggestions / autocomplete
- Index sync on product/inventory updates (via Kafka)
- Sponsored product injection into search results (from ads-service)
- Recently viewed & related product lookups

---

### 10. notification-service — **NestJS**

**Why NestJS:** WebSocket gateway, Bull queue for async email/SMS jobs, and third-party adapters (SendGrid, Twilio) are all well-supported in the NestJS ecosystem.

**Features covered:**

- Email notifications (order placed, shipped, delivered, payment confirmed)
- SMS notifications (OTP, order updates)
- In-app notifications (real-time via WebSocket)
- Push notifications (mobile)
- Low stock alert emails to sellers
- Marketing email campaigns (triggered by marketing-service events)
- Notification preferences (user opt-in/out per channel)

---

### 11. review-service — **NestJS**

**Why NestJS:** Moderate business logic with clear relational data. TypeORM handles review-to-product and review-to-user relations cleanly. class-validator enforces rating bounds and content rules.

**Features covered:**

- Submit product review (text + star rating + images)
- Edit / delete own review
- Seller response to review
- Helpful votes on reviews
- Review moderation (admin flag/remove)
- Aggregate rating calculation per product
- Q&A — buyer questions, seller / community answers
- Review summary (verified purchase badge)

---

### 12. recommendation-service — **NestJS**

**Why NestJS:** Recommendation logic is complex, event-driven business logic — consuming purchase, view, and rating events from Kafka to continuously refine user preference models. NestJS's modular DI container keeps each algorithm (collaborative filtering, content similarity, cold-start fallback) as an injectable, independently testable service. Bull queues handle scheduled model retraining asynchronously. The JS ecosystem provides adequate statistical primitives (`ml-matrix`, cosine similarity, trending scorers) at portfolio scale.

**Features covered:**

- Personalized product recommendations (based on purchase + browsing history)
- "Customers also bought" (collaborative filtering)
- "Related products" (content-based, category/tag similarity)
- Recently viewed products feed
- Homepage feed personalization
- Trending products (time-windowed popularity scoring)

---

### 13. analytics-service — **Go**

**Why Go:** Analytics is a high-throughput Kafka consumption problem — every `order.completed`, `order.cancelled`, and payment event flows here and must be aggregated across thousands of sellers in real time. Go's goroutines handle concurrent event consumption and parallel report computation with minimal memory overhead. Complex GROUP BY queries, window functions, and CTEs are written as raw SQL via `pgx`, avoiding ORM limitations. Scheduled report generation runs as goroutines on ticker intervals — no external task queue needed.

**Features covered:**

- Seller sales reports (daily, weekly, monthly revenue)
- Seller performance metrics (order volume, return rate, delivery score, rating trend)
- Platform-wide GMV and order volume dashboards (admin)
- Top-selling products report
- Buyer behavior analytics (funnel: view → cart → purchase)
- Coupon / promo effectiveness reports
- Affiliate / referral conversion tracking

---

### 14. seller-service — **NestJS**

**Why NestJS:** The seller service is workflow-driven — KYC verification has a clear state machine (pending → under_review → approved/rejected) that maps directly onto NestJS Guards and state transition logic. class-validator enforces complex input rules on KYC document submissions and payout configurations. TypeORM handles the relational data across seller profiles, documents, and performance metrics. This service is operations-heavy, not throughput-heavy, so there is no case for Go.

**Features covered:**

- Seller registration & profile management
- Seller verification status tracking
- Seller product listing management (links to product-service)
- Seller order fulfillment dashboard (view & update order status)
- Payout settings (bank account, payout schedule)
- Seller performance score display
- Seller-to-buyer messaging (links to messaging-service)

---

### 15. media-service — **Go**

**Why Go:** Streaming multipart file uploads with no GC pauses. Direct chunked transfer to MinIO (S3-compatible) without buffering the full file in memory.

**Features covered:**

- Product image upload & storage
- User avatar upload
- Seller banner / logo upload
- File validation (type, size limits)
- CDN URL generation per asset
- Image resizing / thumbnail generation
- File deletion on product/user removal

---

### 16. admin-bff — **NestJS**

**Why NestJS:** A BFF is fundamentally API orchestration — calling 10 downstream services in parallel and composing their responses into a single shaped payload for the admin-dashboard frontend. NestJS's `Promise.all` handles concurrent service calls cleanly, Guards enforce the 6-role permission model at every route declaratively, and typed DTOs catch mismatched aggregations at compile time. TypeScript's type system makes the composition layer safe and maintainable.

**Features covered:**

- User management (view, ban, role assignment)
- Seller approval & suspension
- Product moderation (approve, remove, flag)
- Category management
- Platform announcement broadcasts
- Dispute resolution oversight
- Audit log viewer
- Coupon & promo management

---

### 17. promo-service — **NestJS**

**Why NestJS:** Coupon and promo logic involves complex validation rules (expiry, usage limits, user eligibility, minimum cart value). NestJS's pipe + guard pattern handles this declaratively.

**Features covered:**

- Coupon / promo code creation & management (admin/seller)
- Code validation at checkout (expiry, usage limit, minimum order)
- Percentage and fixed-amount discounts
- User-specific and public coupons
- Flash Deal / Lightning Deal time-window configuration
- Deal activation / deactivation scheduling
- Usage tracking and redemption history

---

### 18. deals-service — **NestJS**

**Why NestJS:** Deal scheduling and time-window enforcement are event-driven — NestJS's built-in scheduler (cron jobs) and event emitter handle activation/expiry cleanly.

**Features covered:**

- Flash Deal creation (product, discount %, time window, quantity cap)
- Lightning Deal queue management
- Deal countdown timer sync (publishes to search-service for badge display)
- Deal activation / expiry events (Kafka → product-service, search-service)
- Deal performance tracking (units sold, revenue during deal)
- Subscribe & Save deal configuration

---

### 19. ads-service — **NestJS**

**Why NestJS:** Ad serving logic (bid management, impression tracking, budget pacing) fits NestJS's modular service pattern. Low-to-moderate throughput for a portfolio system.

**Features covered:**

- Sponsored product listing creation (seller self-serve)
- Bid management per keyword / category
- Daily budget cap enforcement
- Ad impression & click tracking
- Ad injection into search results and category pages
- Seller ad performance dashboard (impressions, clicks, spend, ROAS)
- Ad campaign activation / pause / end

---

### 20. referral-service — **NestJS**

**Why NestJS:** Referral tracking is event-driven and CRUD-moderate. NestJS integrates cleanly with auth-service (user registration events) and payment-service (reward crediting).

**How the affiliate system works:**

Affiliate is NOT a separate user role in the auth-service. Auth-service only manages three roles: `buyer`, `seller`, `admin`. Affiliate is a **program membership** — any buyer can apply, and once approved, referral-service creates an `affiliate_profile` record linked to their `user_id`. This keeps auth-service clean and the entire affiliate domain encapsulated inside referral-service.

**Full affiliate flow:**

```
1. Buyer applies       → POST /referral/affiliate/apply
2. Admin approves      → PATCH /referral/affiliate/:id/approve  (via admin-bff)
3. Profile created     → affiliate_profile { affiliate_id, commission_rate, tier }
4. Affiliate shares    → zamazon.com/p/123?ref=AFFILIATE_ID
5. Visitor clicks      → api-gateway stores ref tag in cookie (30-day window)
6. Visitor buys        → order-service reads ref tag, emits order.placed { affiliate_id }
7. Commission credited → referral-service consumes event, updates earnings ledger
8. Affiliate paid out  → payment-service transfers commission to bank
```

**Database tables owned by referral-service:**

- `affiliate_applications` — (user_id, status, applied_at, reviewed_by)
- `affiliate_profiles` — (user_id, affiliate_id, commission_rate, tier, status)
- `affiliate_clicks` — (affiliate_id, product_id, ip, user_agent, clicked_at)
- `affiliate_conversions` — (affiliate_id, order_id, commission_amount, status, paid_at)

**Features covered:**

- Affiliate program application & approval flow
- Unique affiliate link generation per approved affiliate
- Click tracking with 30-day attribution window
- Commission calculation on order conversion
- Affiliate earnings ledger
- Tiered commission rates (Bronze / Silver / Gold affiliate)
- Affiliate dashboard (clicks, conversions, earnings, payout history)
- Casual referral program (any user, no approval — earn store credit per signup)
- Referral fraud prevention (same-device / same-IP detection, links to fraud-service)

---

### 21. messaging-service — **NestJS**

**Why NestJS:** Real-time buyer-seller chat is a natural fit for NestJS's WebSocket gateway. Message persistence in PostgreSQL; live delivery via Socket.IO.

**Scope boundary:** This service handles P2P transactional chat only — buyer talking to seller about an order or product. AI Q&A and admin support tickets live in support-service. Keeping them separate means the real-time chat system is not coupled to LLM latency or ticket workflows.

**Features covered:**

- Buyer-to-seller direct messaging (per order or product inquiry)
- Real-time message delivery (WebSocket)
- Message history & thread view
- Unread message badge count
- File / image sharing in chat
- Message moderation (admin can review flagged threads)
- Auto-reply templates for sellers

---

### 21b. support-service — **NestJS**

**Why NestJS:** Support tickets are workflow-driven CRUD (open → assigned → resolved). NestJS's modular structure handles the AI integration cleanly — the LLM call is just another injectable service. Bull queue manages async AI response generation.

**Why separate from messaging-service:** messaging-service is real-time P2P chat (WebSocket). support-service is async ticket/Q&A workflows with LLM calls that can take seconds. Mixing them would couple real-time chat latency to AI inference time.

**AI Q&A architecture (RAG pattern):**

```
Buyer types a product question
→ support-service fetches product details + reviews from product-service / review-service
→ builds context window: [product spec, reviews, seller FAQ]
→ sends context + question to LLM (e.g. Claude API)
→ LLM answers grounded in product knowledge base
→ if confidence score < threshold → "Route to seller" or open ticket
```

**Features covered:**

- AI chatbot for product Q&A (RAG on product catalog + reviews)
- AI confidence scoring — low-confidence answers escalate to seller or ticket
- Buyer → admin support ticket (order issues, account problems, complaints)
- Seller → admin support ticket (payout issues, listing disputes, policy questions)
- Ticket status lifecycle (open → assigned → in-progress → resolved → closed)
- Ticket priority and category tagging
- Admin ticket assignment and response
- Escalation rules (auto-escalate after N days with no response)
- Support chat history and ticket audit trail

---

### 22. loyalty-service — **NestJS**

**Why NestJS:** Points accumulation and redemption logic is rule-based and event-driven. NestJS event listeners on order.completed and payment.confirmed make this straightforward.

**Features covered:**

- Points earning on every purchase (configurable rate)
- Points redemption at checkout (partial payment)
- Points expiry management
- Loyalty tier management (Bronze, Silver, Gold, Prime-equivalent)
- Tier-based perks (free shipping threshold, early deal access)
- Points transaction history
- Loyalty program dashboard for buyers

---

### 23. dispute-service — **NestJS**

**Why NestJS:** Dispute management is workflow-driven (open → under review → resolved). NestJS's state machine pattern and TypeORM handle this lifecycle cleanly.

**Features covered:**

- Dispute / claim creation by buyer (wrong item, not delivered, damaged)
- Evidence upload (links to media-service)
- Seller response to dispute
- Admin arbitration workflow
- Resolution actions (refund, replacement, rejection)
- Dispute status tracking & notifications
- Escalation rules (auto-escalate after N days of no response)

---

### 24. fraud-service — **Go**

**Why Go:** Fraud signal processing needs to be fast and non-blocking. Go handles concurrent rule evaluation and real-time scoring without adding latency to the payment or order flow.

**Features covered:**

- Real-time fraud scoring on payment attempts
- Velocity checks (too many orders/payments in short window)
- Suspicious IP / device fingerprint flagging
- Account takeover detection (unusual login location/device)
- Seller fraud detection (fake reviews, review manipulation)
- Referral fraud detection (same-device multi-account abuse)
- Fraud alert emission (blocks payment or flags order for manual review)

---

### 25. audit-log-service — **Go**

**Why Go:** The audit log is a pure write-heavy Kafka consumer. Every significant action across the platform produces an event; audit-log-service subscribes to all of them and persists an immutable record. Go's lightweight goroutines handle high consumer throughput with minimal memory overhead.

**Why a dedicated service:** Audit logs must be immutable and always available, even if the originating service is down. Centralizing them in one service means one place to query, one retention policy, and one security boundary. Individual services should not own their own audit storage.

**Features covered:**

- Consume all significant Kafka events across every service
- Write immutable audit records (who, what, when, result, source IP)
- Indexed by: user_id, resource_type, resource_id, timestamp
- Admin audit trail viewer (via admin-bff)
- Tamper detection (hash chaining on records)
- Retention policy enforcement (archive older records to cold storage)
- Compliance export (downloadable audit report for a user or time range)

---

## Design Decisions

### User profile — kept inside user-service

Not split into a separate service. The meaningful separation is already: auth-service (identity + tokens) vs user-service (profile + addresses + wishlist). Splitting profile further adds inter-service hops with no benefit at this scale.

### Seller profile — kept inside seller-service

Same reasoning. seller-service owns both the seller's account metadata and their operational data (listings, payouts, performance). A separate seller-profile-service would just be a thin CRUD wrapper.

### Cache — not a service

Redis is shared infrastructure, not a microservice. Each service connects to Redis directly and manages its own key namespace. No "cache service" layer is needed.

### Withdraw — not a separate service

Withdrawal requests are created and tracked inside wallet-service. The actual bank transfer execution is a call from wallet-service → payment-service. A separate withdraw-service would be a one-feature stub with no justification.

### Report — not a separate service

All reporting lives in analytics-service (Go). Raw SQL via `pgx` handles complex aggregation well and goroutines parallelize report generation across sellers. A report-service would duplicate analytics-service with no added value.

---

## Infrastructure Stack

| Concern             | Tool                                            |
| ------------------- | ----------------------------------------------- |
| Message broker      | Apache Kafka                                    |
| Cache               | Redis                                           |
| Relational DB       | PostgreSQL (users, orders, payments, inventory) |
| Document DB         | MongoDB (product catalog)                       |
| Search index        | Elasticsearch                                   |
| Object storage      | MinIO (S3-compatible)                           |
| Containers (dev)    | Docker + Docker Compose                         |
| Containers (prod)   | Kubernetes                                      |
| Service discovery   | Kubernetes DNS                                  |
| Monitoring          | Prometheus + Grafana                            |
| Distributed tracing | OpenTelemetry + Jaeger                          |

---

## Inter-service Communication

**Synchronous (request/response)**

- REST — all client-facing APIs through the API Gateway
- gRPC — internal service-to-service calls

**Asynchronous (event-driven via Kafka)**

| Topic                         | Producer          | Consumers                                                                                   |
| ----------------------------- | ----------------- | ------------------------------------------------------------------------------------------- |
| `order.placed`                | order-service     | inventory-service, notification-service, loyalty-service, audit-log-service                 |
| `order.completed`             | order-service     | loyalty-service, analytics-service, wallet-service, audit-log-service                       |
| `order.cancelled`             | order-service     | inventory-service, payment-service, notification-service, wallet-service, audit-log-service |
| `payment.confirmed`           | payment-service   | order-service, notification-service, fraud-service, audit-log-service                       |
| `payment.initiated`           | payment-service   | fraud-service, audit-log-service                                                            |
| `payment.failed`              | payment-service   | order-service, notification-service, audit-log-service                                      |
| `wallet.credited`             | wallet-service    | notification-service, audit-log-service                                                     |
| `wallet.withdrawal.requested` | wallet-service    | payment-service, audit-log-service                                                          |
| `wallet.withdrawal.completed` | payment-service   | wallet-service, notification-service, audit-log-service                                     |
| `inventory.low`               | inventory-service | notification-service                                                                        |
| `inventory.updated`           | inventory-service | product-service, search-service                                                             |
| `user.registered`             | auth-service      | user-service, notification-service, referral-service, audit-log-service                     |
| `notification.send`           | any service       | notification-service                                                                        |
| `deal.activated`              | deals-service     | product-service, search-service                                                             |
| `deal.expired`                | deals-service     | product-service, search-service                                                             |
| `dispute.opened`              | dispute-service   | notification-service, audit-log-service                                                     |
| `fraud.flagged`               | fraud-service     | payment-service, order-service, audit-log-service                                           |
| `support.ticket.created`      | support-service   | notification-service, audit-log-service                                                     |
| `support.ticket.resolved`     | support-service   | notification-service, audit-log-service                                                     |

---

## Frontend Architecture

Three separate Next.js 16 applications — each is an independent app in the monorepo. All talk to the api-gateway only — no direct service calls from any frontend.

| App             | Tech                    | Port | Users   | Rendering                                                                         |
| --------------- | ----------------------- | ---- | ------- | --------------------------------------------------------------------------------- |
| customer-web    | Next.js 16 (App Router) | 3000 | Buyers  | SSR + Server Components for product/category pages; CSR for cart/checkout/account |
| seller-portal   | Next.js 16 (App Router) | 3001 | Sellers | Mostly CSR — no SEO needed; App Router used for nested dashboard layouts          |
| admin-dashboard | Next.js 16 (App Router) | 3002 | Admins  | Mostly CSR — internal tool; App Router for role-scoped nested layouts             |

**Why Next.js 16 for all three:**

- App Router nested layouts give seller and admin dashboards a free persistent sidebar + sub-layouts per section without manual wiring
- Consistent tooling across the monorepo (one framework, one build system)
- Server Components reduce bundle size for data-heavy dashboard pages
- seller-portal and admin-dashboard use `'use client'` where interactivity is needed — Next.js does not force SSR

**customer-web (Next.js 16, :3000)**

- SSR / Server Components: `/`, `/products/[id]`, `/category/[slug]`, `/search` — SEO-critical
- CSR: `/cart`, `/checkout`, `/account`, `/orders/[id]`
- Real-time: notification bell (WebSocket), buyer-seller chat (messaging-service)
- Auth: JWT in HttpOnly cookie
- State: React Query (server state) + Zustand (client UI state)

**seller-portal (Next.js 16, :3001)**

- Sections: Dashboard, Products, Inventory, Orders, Analytics, Payouts, Ads, Messages
- Real-time: new order alerts, low-stock notifications (WebSocket)
- Auth: JWT in HttpOnly cookie — only `seller` role can access
- State: React Query + Zustand

**admin-dashboard (Next.js 16, :3002)**

- Three separate apps share the same Next.js process; admin is internal-only
- Auth: JWT in HttpOnly cookie — only `admin` role can access
- Route visibility is controlled by the user's **permission set** (not just role name)
- Backend also enforces permissions on every API call — frontend hiding is UX only

### Admin Role & Permission Model

Admin is not one flat role. Six scoped roles with explicit permission sets:

| Admin Role      | Permissions                                          | Access                                                |
| --------------- | ---------------------------------------------------- | ----------------------------------------------------- |
| `super_admin`   | `*` (all)                                            | Everything                                            |
| `operations`    | `users:rw`, `sellers:rw`, `orders:rw`, `disputes:rw` | Users, sellers, orders, disputes                      |
| `moderator`     | `products:moderate`, `reviews:moderate`              | Product/review approval and removal                   |
| `finance`       | `payments:read`, `payouts:rw`, `analytics:financial` | Payouts, refunds, financial reports                   |
| `support_agent` | `tickets:rw`, `users:read`, `orders:read`            | Support tickets + limited lookup                      |
| `monitor`       | `analytics:read`, `monitoring:read`                  | Read-only dashboards and system health — no mutations |

**How it works:**

```
JWT payload (admin user):
{
  "sub": "admin_456",
  "role": "monitor",
  "permissions": ["analytics:read", "monitoring:read"]
}
```

- `auth-service` issues the JWT with role + permissions on login
- `api-gateway` validates permissions on every inbound request before routing
- `admin-bff` validates again before calling downstream services (defense in depth)
- `admin-dashboard` reads permissions from JWT to show/hide routes (UX layer only)

**Admin route access map:**

| Route               | Minimum permission required     |
| ------------------- | ------------------------------- |
| `/admin/dashboard`  | `monitoring:read`               |
| `/admin/analytics`  | `analytics:read`                |
| `/admin/users`      | `users:read`                    |
| `/admin/sellers`    | `sellers:read`                  |
| `/admin/products`   | `products:moderate`             |
| `/admin/orders`     | `orders:read`                   |
| `/admin/payouts`    | `payouts:read`                  |
| `/admin/disputes`   | `disputes:read`                 |
| `/admin/support`    | `tickets:read`                  |
| `/admin/audit-logs` | `audit:read` (super_admin only) |
| `/admin/system`     | `monitoring:read`               |

**CDN**

- Cloudflare CDN sits in front of customer-web static assets and media-service
- Product images served via CDN edge, not directly from MinIO

---

## Folder Structure (target)

```
zamazon/
├── idea.md
├── prd/
│   ├── overview.md
│   ├── 01-api-gateway.md
│   ├── 02-auth-service.md
│   ├── 03-user-service.md
│   ├── 04-product-service.md
│   ├── 05-inventory-service.md
│   ├── 06-search-service.md
│   ├── 07-media-service.md
│   ├── 08-cart-service.md
│   ├── 09-order-service.md
│   ├── 10-payment-service.md
│   ├── 11-wallet-service.md
│   ├── 12-notification-service.md
│   ├── 13-review-service.md
│   ├── 14-recommendation-service.md
│   ├── 15-deals-service.md
│   ├── 16-promo-service.md
│   ├── 17-loyalty-service.md
│   ├── 18-messaging-service.md
│   ├── 19-support-service.md
│   ├── 20-seller-service.md
│   ├── 21-ads-service.md
│   ├── 22-referral-service.md
│   ├── 23-analytics-service.md
│   ├── 24-admin-bff.md
│   ├── 25-dispute-service.md
│   ├── 26-fraud-service.md
│   └── 27-audit-log-service.md
├── srs/
│   ├── overview.md
│   └── [one file per service — written after PRD approved]
├── apps/
│   ├── customer-web/        Next.js 14
│   ├── seller-portal/       React + Vite
│   └── admin-dashboard/     React + Vite
└── services/
    ├── api-gateway/
    ├── auth-service/
    ├── user-service/
    ├── product-service/
    ├── inventory-service/
    ├── search-service/
    ├── media-service/
    ├── cart-service/
    ├── order-service/
    ├── payment-service/
    ├── wallet-service/
    ├── notification-service/
    ├── review-service/
    ├── recommendation-service/
    ├── deals-service/
    ├── promo-service/
    ├── loyalty-service/
    ├── messaging-service/
    ├── support-service/
    ├── seller-service/
    ├── ads-service/
    ├── referral-service/
    ├── analytics-service/
    ├── admin-bff/
    ├── dispute-service/
    ├── fraud-service/
    ├── wallet-service/
    └── audit-log-service/
```

---

## Next Step

Write `docs/prd/` — start with `overview.md` (personas, goals, scope) then one file per user type.
