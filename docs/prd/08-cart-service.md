# Cart Service

**Technology:** NestJS
**Port:** 8004
**Priority:** P1 — Core Platform

---

## Why NestJS

The shopping cart lives primarily in Redis — a fast in-memory store — and needs to react to real-time events like price changes and stock updates. NestJS has a first-class Redis integration module and an event-driven architecture that makes syncing the cart with live product data clean and straightforward. The service also needs to coordinate with promo-service for coupon validation, which fits naturally into NestJS's injectable service pattern. Go would work but has no equivalent Redis client ecosystem for this pattern; Django would be heavier than needed for a session-focused service.

---

## What This Service Does

- Allows buyers to add products to their cart, update quantities, and remove items freely before checkout
- Supports guest carts — browsers without an account can still build a cart, and when the guest logs in, their cart is seamlessly merged with any existing cart from their account
- Continuously recalculates cart totals to reflect the current live price of each item — if a seller changes a price or a deal expires, the buyer sees the updated total immediately
- Allows buyers to apply coupon or promo codes to their cart — the service validates the code with promo-service and shows the discount applied
- Provides the cart item count badge that updates in real time in the navigation header
- Lets buyers save items for later — moving them out of the active cart but keeping them easily accessible without adding them to a wishlist

---

## Who Uses It

- **Buyers** — the primary cart experience from adding the first item through to proceeding to checkout
- **order-service** — reads the cart contents when the buyer initiates checkout
- **promo-service** — validates coupon codes applied to the cart

---

## Key Integrations

- Reads current product prices from product-service to keep cart totals accurate
- Validates coupon codes with promo-service before applying discounts
- Passes the finalised cart to order-service when the buyer proceeds to checkout
- Reacts to price change events so the displayed total is never stale
