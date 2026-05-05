# Deals Service

**Technology:** NestJS
**Port:** 8007
**Priority:** P2 — Enhanced Buyer Experience

---

## Why NestJS

Flash Deals and Lightning Deals are time-driven — they activate at a scheduled moment, run for a fixed window, and expire automatically. NestJS has a built-in task scheduler (cron jobs) and event emitter that handle this lifecycle cleanly without external tooling. The event-driven nature of deal activation and expiry — which must notify search-service and product-service in real time — fits naturally into NestJS's Kafka integration. Go would work but lacks the scheduler ecosystem.

---

## What This Service Does

- Allows sellers to create Flash Deals — a product is offered at a discounted price for a defined time window (e.g. 20% off for 4 hours), with an optional cap on total units sold at the deal price
- Manages a Lightning Deal queue — short-window deals (e.g. 30 minutes) are reviewed and approved by admins before going live, similar to Amazon's Lightning Deals
- Activates deals automatically at their scheduled start time and deactivates them at expiry — no manual intervention required
- Broadcasts deal countdown timers to product pages and search results so buyers can see exactly how much time is left on a deal
- Enforces unit caps — once the deal quantity limit is reached, the deal price is no longer available even if time remains
- Tracks deal performance in real time: units sold, revenue generated, and conversion rate during the deal window
- Supports Subscribe & Save deal configuration — sellers can define the recurring discount percentage for products enrolled in the Subscribe & Save programme
- Sends deal expiry and activation notifications to relevant services via Kafka so product listings and search results update immediately

---

## Who Uses It

- **Sellers** — create and schedule Flash Deals and set Subscribe & Save discounts for their products
- **Admins** — review and approve Lightning Deal submissions before they go live
- **Buyers** — see deal badges, countdown timers, and deal prices on product pages and search results
- **search-service** — receives activation and expiry events to add or remove deal badges on results

---

## Key Integrations

- Publishes `deal.activated` and `deal.expired` Kafka events consumed by product-service and search-service
- Reads product and pricing data from product-service to validate deal configurations before approval
- Notifies order-service of active deals so deal pricing is validated at the moment of purchase
