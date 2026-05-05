# Analytics Service

**Technology:** Go
**Port:** 8202
**Priority:** P4 — Platform Operations

---

## Why Go

Analytics is fundamentally a high-throughput Kafka consumption problem — every `order.completed`, `order.cancelled`, and payment event across the platform flows through this service and must be aggregated in real time. Go's goroutine model handles concurrent event consumption and parallel report computation efficiently without the memory overhead of a Node.js runtime. Complex GROUP BY aggregations are written as raw SQL via `pgx`, which gives full control and avoids ORM limitations on window functions and CTEs. Scheduled report generation runs as goroutines with ticker-based intervals — no external task queue needed. The same performance profile that makes Go the right choice for order-service and payment-service applies here.

---

## What This Service Does

**For Sellers**
- Delivers a daily, weekly, and monthly sales report showing revenue, order volume, average order value, and top-selling products
- Tracks seller performance metrics over time: average star rating trend, on-time delivery rate, cancellation rate, and return rate — the same metrics that affect their visibility ranking
- Shows ad campaign performance metrics alongside organic sales so sellers can understand total ROI
- Provides coupon and deal effectiveness reports — how much discount was distributed and how much revenue each promotion generated

**For Admins**
- Tracks platform-wide Gross Merchandise Value (GMV) and order volume at daily, weekly, and monthly granularity
- Shows the buyer conversion funnel: how many visitors viewed products, added to cart, reached checkout, and completed a purchase — and where buyers are dropping off
- Provides a top-selling products report by category and time period
- Tracks affiliate programme performance: total clicks, conversions, commission paid, and top-performing affiliates
- Shows platform revenue from commissions, ad spend, and subscription fees over time

---

## Who Uses It

- **Sellers** — view their own store performance on the seller portal analytics section
- **Admins (finance, operations, monitor roles)** — view platform-wide business metrics on the admin dashboard
- **seller-service** — reads seller performance scores calculated here

---

## Key Integrations

- Consumes `order.completed` and `order.cancelled` events from order-service as the primary data source
- Reads ad performance data from ads-service
- Reads coupon redemption data from promo-service
- Reads affiliate conversion data from referral-service
- Provides seller performance scores to seller-service for ranking and display
