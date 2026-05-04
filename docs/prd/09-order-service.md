# Order Service

**Technology:** Go
**Port:** 8102
**Priority:** P1 — Core Platform

---

## Why Go

Placing an order is a distributed transaction — it must atomically reserve stock, charge payment, confirm the order, and notify everyone involved, all while guaranteeing that a failure at any step rolls back cleanly without leaving the system in an inconsistent state. This pattern is called a Saga and it requires precise, explicit control over every success and failure path. Go's error handling forces every failure case to be handled explicitly, making it the right fit for the most financially critical flow on the platform. A bug here means overselling, double-charging, or lost orders — the cost of getting it wrong is too high for a runtime with unpredictable behavior.

---

## What This Service Does

- Accepts a buyer's cart and converts it into a confirmed order with a unique order reference number
- Handles orders containing products from multiple sellers — splits them into per-seller fulfilment units while presenting the buyer with a single unified order
- Manages the full order status lifecycle from placement through to completion: Pending → Confirmed → Processing → Shipped → Delivered → Completed
- Allows buyers to cancel orders within the permitted cancellation window — triggers stock release and refund automatically
- Lets buyers re-order any past order by cloning it directly into their current cart
- Supports Subscribe & Save — recurring orders are automatically scheduled and processed at the buyer's chosen interval without requiring manual reorder
- Generates an invoice for every completed order, downloadable by the buyer from their order history
- Enforces Flash Deal and Lightning Deal time-window rules — a deal price is only honoured if the order is placed within the active deal window
- Allows buyers to initiate a dispute from within an order — connects to dispute-service

---

## Who Uses It

- **Buyers** — place orders, track status, cancel, re-order, access invoices
- **Sellers** — view incoming orders, update fulfilment status (processing → shipped → delivered)
- **Admins** — review and manage orders with elevated access
- **payment-service** — called to charge the buyer after stock is reserved
- **inventory-service** — called to reserve and then confirm stock deduction

---

## Key Integrations

- Calls inventory-service to reserve stock before payment
- Calls payment-service to charge the buyer after reservation is confirmed
- Publishes `order.placed`, `order.completed`, and `order.cancelled` events consumed by loyalty-service, analytics-service, wallet-service, notification-service, and audit-log-service
- Reads cart from cart-service at checkout initiation
- Links disputes created here to dispute-service
