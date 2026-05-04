# Loyalty Service

**Technology:** NestJS
**Port:** 8009
**Priority:** P2 — Enhanced Buyer Experience

---

## Why NestJS

The loyalty system is entirely event-driven — points are earned and expired in response to things that happen elsewhere on the platform (orders completed, payments confirmed, time passing). NestJS's event listener pattern lets this service subscribe to those events and respond without being called directly by other services. The rules engine for calculating points, managing tiers, and applying perks is business logic-heavy and well-suited to NestJS's injectable service and decorator patterns.

---

## What This Service Does

- Awards points to buyers automatically every time they complete a purchase — the points rate is configurable (e.g. 1 point per dollar spent)
- Allows buyers to redeem accumulated points as partial or full payment at checkout — points are converted to monetary value at a defined rate
- Manages point expiry — points earned more than a configured number of months ago are automatically expired to encourage active use
- Organises buyers into loyalty tiers based on their total lifetime points: Bronze, Silver, Gold, and Prime (top tier) — tiers are recalculated monthly
- Unlocks tier-based perks that become more valuable at higher tiers — free shipping on all orders, early access to Lightning Deals, higher referral commission rates, priority customer support
- Gives buyers a complete, chronological transaction history of every point earned, spent, and expired, with the reason for each event
- Shows buyers their current tier, points balance, and progress toward the next tier on their account dashboard
- Notifies buyers when they earn points, when points are about to expire, and when they reach a new tier

---

## Who Uses It

- **Buyers** — earn points on purchases, redeem at checkout, view tier status and perks
- **order-service** — loyalty points are validated and deducted when used as checkout payment
- **notification-service** — sends tier-up congratulations and expiry warning notifications

---

## Key Integrations

- Consumes `order.completed` from order-service to credit points to the buyer's balance
- Consumes `order.cancelled` from order-service to reverse points if an order is cancelled after earning
- Provides point balance and tier data to cart-service so buyers can see points available to redeem during checkout
- Publishes tier change events to notification-service so buyers are informed when they reach a new tier
