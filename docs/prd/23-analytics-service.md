# Analytics Service

**Technology:** Django
**Port:** 8202
**Priority:** P4 — Platform Operations

---

## Why Django

Analytics means aggregating large volumes of data into meaningful summaries — totals by day, trends over weeks, rankings by category. Python's pandas library is the best tool available for this type of computation, and it integrates directly into Django. Celery (Python's async task queue) handles scheduled report generation without blocking API responses. The reporting queries in this service are complex GROUP BY aggregations that would be verbose and error-prone to write in Go or NestJS without ORM support.

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
