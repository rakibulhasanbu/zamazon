# Inventory Service

**Technology:** Go
**Port:** 8101
**Priority:** P1 — Core Platform

---

## Why Go

Inventory is the most concurrency-sensitive service on the platform. When a product has one unit left in stock and two buyers click "Buy Now" at exactly the same moment, the system must guarantee that only one of them succeeds and the other sees "out of stock" — no overselling. Go's concurrency model and low-level database control allow atomic stock operations that are impossible to get wrong. Node.js's single-threaded event loop and Python's GIL make this level of concurrent correctness much harder to guarantee safely.

---

## What This Service Does

- Tracks the exact stock level for every product variant across every warehouse location
- When a buyer proceeds to checkout, the service atomically reserves the required quantity — it is held for that buyer's session and cannot be sold to anyone else during the checkout window
- Releases reserved stock back to available if the buyer abandons checkout, the payment fails, or the order is cancelled
- Permanently deducts stock when an order is confirmed and paid
- Detects when any product's stock falls below a configurable threshold and triggers a low-stock alert to notify the seller automatically
- Maintains a full history of every stock adjustment — automatic (from orders) and manual (seller corrections) — so sellers can audit discrepancies
- Supports multi-warehouse stock management — the same product can be stocked across multiple locations, and the service aggregates total available quantity for display

---

## Who Uses It

- **Sellers** — view real-time stock levels, make manual adjustments, receive low-stock alerts
- **order-service** — calls this service to reserve and confirm stock during checkout
- **product-service** — reads stock levels to show availability status on product pages
- **notification-service** — receives low-stock events to alert sellers

---

## Key Integrations

- Receives stock reservation requests from order-service during checkout
- Publishes `inventory.low` events when stock drops below threshold — notification-service sends alerts to sellers
- Publishes `inventory.updated` events so product-service and search-service can reflect current availability
- Releases reserved stock automatically when order-service publishes `order.cancelled`
