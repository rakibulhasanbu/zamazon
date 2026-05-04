# Payment Service

**Technology:** Go
**Port:** 8103
**Priority:** P1 — Core Platform

---

## Why Go

Payment processing has zero tolerance for ambiguity. When a charge fails, the system must know exactly why and handle that specific failure case correctly — not guess, not retry blindly. Go's explicit error handling makes every failure path visible and deliberate. There are no garbage collection pauses that could cause a timeout mid-transaction, no hidden exceptions that could leave a charge in an unknown state. Financial operations need the most predictable runtime available, and Go delivers that.

---

## What This Service Does

- Integrates with multiple external payment gateways — Stripe, PayPal, and others — so buyers can pay using their preferred method
- Stores payment methods securely using tokenisation — the platform never stores raw card numbers, only a token that represents the card with the payment gateway
- Processes payment capture when an order is confirmed — charges the buyer's selected payment method for the exact order amount
- Handles full and partial refunds when orders are cancelled, items are returned, or disputes are resolved in the buyer's favour
- Processes bank transfer payouts to sellers when wallet-service triggers a withdrawal request — the earnings move from the platform to the seller's bank account
- Handles asynchronous payment callbacks from gateways — a payment doesn't always succeed or fail instantly, and this service listens for the gateway's final confirmation webhook
- Passes payment attempt signals to fraud-service for real-time risk scoring before processing
- Emits clear outcome events (`payment.confirmed` or `payment.failed`) so order-service and notification-service can respond appropriately

---

## Who Uses It

- **Buyers** — transparent — they initiate payment through the checkout flow, this service handles the gateway interaction
- **Sellers** — receive payouts processed by this service when they request withdrawal
- **order-service** — calls this service to charge buyers during checkout
- **wallet-service** — calls this service to execute bank transfer payouts

---

## Key Integrations

- Receives payment requests from order-service during checkout
- Calls fraud-service before processing any payment for risk scoring
- Executes payout transfers triggered by wallet-service withdrawal requests
- Publishes `payment.confirmed`, `payment.initiated`, and `payment.failed` events consumed by order-service, notification-service, fraud-service, wallet-service, and audit-log-service
