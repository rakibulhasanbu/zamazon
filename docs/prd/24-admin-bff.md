# Admin BFF (Backend for Frontend)

**Technology:** NestJS
**Port:** 8204
**Priority:** P4 — Platform Operations

---

## Why NestJS

A BFF is fundamentally an API orchestration layer — it calls multiple downstream services in parallel and composes their responses into a single shaped payload for the admin-dashboard frontend. NestJS excels at this: `Promise.all` for concurrent service calls, Guards for permission enforcement at every route, and typed DTOs for each aggregated response (user detail = profile + orders + wallet + disputes + tickets). The admin permission model (6 scoped roles, each with explicit permission arrays) maps directly onto NestJS Guard decorators, keeping authorization logic declarative and auditable. TypeScript's type system catches mismatched aggregations at compile time.

---

## What This Service Does

- Aggregates data from multiple backend services into single, composed responses for the admin dashboard — the admin never waits for the frontend to make five separate API calls to see one user's complete record
- Manages user accounts: view buyer profiles, order history, wallet balance, dispute history, and support tickets — admins with the correct permissions can suspend, ban, or restore accounts
- Manages seller accounts: view seller profiles, product listings, performance metrics, payout history, and KYC documents — admins can approve, suspend, or permanently remove sellers
- Provides product moderation tools: view flagged listings, approve new products from pending sellers, remove policy-violating content, manage the category tree and product attributes
- Manages the dispute resolution queue: view all open disputes, read the evidence submitted by both parties, and issue a resolution decision
- Provides access to the audit log viewer: admins with `audit:read` permission can search the immutable event history by user, resource, or time range
- Sends platform-wide announcements — admins can broadcast a message that appears in all users' notification centres
- Manages coupon and promotion codes at the platform level
- Enforces the admin permission model — every response is filtered based on the requesting admin's permission set, and operations that exceed the admin's permissions are rejected

---

## Who Uses It

- **Admins (all roles)** — this is the only backend the admin-dashboard talks to; it enforces which data each admin role can access
- Translates admin actions into calls to the appropriate downstream services

---

## Key Integrations

- Aggregates data from user-service, order-service, wallet-service, dispute-service, analytics-service, audit-log-service, product-service, seller-service, and support-service
- Enforces the permission model defined in auth-service — every admin request is validated against the JWT permission scopes before data is returned
- All admin-initiated actions (ban user, approve seller, resolve dispute) are forwarded to the appropriate service and logged to audit-log-service
- Routes seller KYC review actions to seller-service (`GET /kyc/submissions` for the review queue, `POST /kyc/review` for decisions) — admin-bff does not own KYC state
