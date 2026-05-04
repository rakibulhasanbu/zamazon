# Support Service

**Technology:** NestJS
**Port:** 8011
**Priority:** P2 — Enhanced Buyer Experience

---

## Why NestJS

This service combines two distinct workloads: real-time AI-powered Q&A (which involves calling an LLM API) and structured ticket management (which is workflow-driven CRUD). NestJS's Bull queue handles AI responses asynchronously — the question is received immediately, the LLM call is queued, and the answer is delivered when ready without blocking other requests. The ticket workflow fits naturally into NestJS's state machine and TypeORM patterns. Keeping this separate from messaging-service is a deliberate decision — LLM response times (1–5 seconds) must never introduce latency into the real-time buyer-seller chat.

---

## What This Service Does

- Provides an AI-powered product assistant on every product page — buyers type a natural language question ("Does this fit a 15-inch laptop?") and receive an answer generated from the product's specifications, descriptions, and existing reviews
- Scores the confidence of each AI answer — if the model is uncertain, the question is automatically escalated to the seller or converted into a visible Q&A post on the product page
- Allows buyers to open a support ticket for platform-level problems that cannot be resolved by the seller: order never arrived, account access issues, payment disputes that need admin intervention, policy complaints
- Allows sellers to open support tickets for operational issues: payout problems, listing restriction appeals, policy questions, account verification delays
- Manages the full support ticket lifecycle: Open → Assigned → In Progress → Resolved → Closed
- Assigns tickets to the appropriate admin team based on the ticket category and priority
- Escalates tickets automatically when they receive no response within a defined time window — escalated tickets are prioritised in the admin queue
- Sends status update notifications to the ticket creator at every stage change

---

## Who Uses It

- **Buyers** — use the AI product assistant to get instant answers, open support tickets for unresolved problems
- **Sellers** — open support tickets for operational and account issues
- **Admins (support_agent role)** — manage and respond to ticket queues

---

## Key Integrations

- Reads product data from product-service and review data from review-service to build context for AI answers (RAG pattern)
- Calls an external LLM API (Claude) for generating product Q&A responses
- Links tickets to orders from order-service when the issue is order-related
- Publishes `support.ticket.created` and `support.ticket.resolved` events consumed by notification-service and audit-log-service
