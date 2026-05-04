# Wallet Service

**Technology:** Go
**Port:** 8104
**Priority:** P1 — Core Platform

---

## Why Go

The wallet is an internal financial ledger — every credit and debit must be recorded atomically with no possibility of a balance going negative or two concurrent redemptions both succeeding when only one should. Go's concurrency primitives allow precise control over these atomic operations. Crucially, this service has no external network dependencies — it only reads and writes to its own database, which means Go's speed advantage is fully utilised without the unpredictability of external API calls. Separating this from payment-service means a Stripe outage never blocks a buyer from checking their wallet balance.

---

## What This Service Does

- Maintains an internal wallet balance for every buyer — this balance can hold store credit, cashback from promotions, and referral rewards
- Maintains an earnings ledger for every seller — when an order is completed, the seller's earnings are credited here minus the platform commission; when an order is refunded, the credit is reversed
- Allows buyers to top up their wallet balance using payment-service (e.g. adding store credit via a card payment)
- Allows buyers to pay for orders fully or partially using their wallet balance at checkout — the balance is locked during checkout and either confirmed (payment succeeded) or released (payment failed)
- Allows sellers and buyers to request a withdrawal — the request is logged here with a status, and payment-service executes the actual bank transfer
- Tracks every single credit and debit with a clear reason code — order completion, refund, promo reward, referral bonus, manual adjustment — so users have a full, auditable transaction history
- Manages the balance lock during checkout — reserved funds cannot be spent elsewhere while an order is in progress

---

## Who Uses It

- **Buyers** — check wallet balance, use balance at checkout, request withdrawals, view transaction history
- **Sellers** — track earnings, request payouts, view earnings history
- **order-service** — reads wallet balance at checkout, confirms or releases locks after payment
- **payment-service** — executes the actual bank transfer when this service approves a withdrawal

---

## Key Integrations

- Consumes `order.completed` to credit seller earnings and any applicable buyer cashback
- Consumes `order.cancelled` to reverse seller earnings and release any buyer balance lock
- Triggers payment-service when a withdrawal request is approved to execute the bank transfer
- Publishes `wallet.credited` and `wallet.withdrawal.requested` events consumed by notification-service and audit-log-service
