# Dispute Service

**Technology:** NestJS
**Port:** 8014
**Priority:** P4 — Platform Operations

---

## Why NestJS

A dispute is a workflow — it moves through defined states (Open → Under Review → Awaiting Seller Response → Escalated → Resolved → Closed) with rules governing the transitions. NestJS's state machine pattern and TypeORM handle these lifecycle transitions cleanly, and the business rules (auto-escalation deadlines, response windows, resolution actions) are expressed clearly as injectable services. The integration with notification-service for status update emails and audit-log-service for compliance logging also fits NestJS's event-driven module pattern.

---

## What This Service Does

- Allows buyers to open a formal dispute directly from their order page when a problem cannot be resolved through buyer-seller messaging — valid reasons include item not received, item significantly different from description, item arrived damaged, and incorrect item sent
- Allows buyers to upload evidence to support their claim — photos, screenshots, and documents stored through media-service
- Notifies the seller of the dispute and gives them a defined window (e.g. 5 business days) to respond with their own evidence and position
- Routes unresolved or escalated disputes to the admin dispute resolution queue where an admin with `disputes:rw` permission reviews both parties' submissions and issues a binding decision
- Escalates disputes automatically when a response deadline passes without action — ensuring no dispute is silently forgotten
- Issues resolution outcomes that trigger concrete actions: full refund to the buyer (via payment-service), partial refund, item replacement request to the seller, or rejection of the claim
- Tracks the full history of every dispute — all messages, evidence uploads, status changes, and resolution decisions — as a permanent record
- Shows buyers and sellers the current status of any open dispute from their respective portals

---

## Who Uses It

- **Buyers** — open disputes, submit evidence, track resolution status
- **Sellers** — receive dispute notifications, submit their response and evidence
- **Admins (operations role)** — review escalated disputes, issue binding resolutions

---

## Key Integrations

- Triggered from order-service when a buyer initiates a dispute on an order
- Calls payment-service to execute refunds when a resolution decision includes a monetary outcome
- Links evidence uploads to media-service for file storage
- Publishes `dispute.opened` events consumed by notification-service and audit-log-service
- Sends resolution outcomes back to order-service to update the order status
