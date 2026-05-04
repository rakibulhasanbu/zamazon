# Promo Service

**Technology:** NestJS
**Port:** 8008
**Priority:** P2 — Enhanced Buyer Experience

---

## Why NestJS

Coupon validation is a multi-rule decision — is the code still active? has the usage limit been reached? does the buyer's cart meet the minimum order value? is the buyer eligible for this specific code? NestJS's Pipe and Guard patterns let each of these rules be composed as reusable, independently testable validators. The service also needs to handle concurrent redemptions safely (two buyers using the same limited-use code at the same moment) which NestJS handles cleanly with database-level locking.

---

## What This Service Does

- Allows admins and sellers to create coupon codes with configurable rules: discount type (percentage or fixed amount), discount value, expiry date, maximum number of uses, and minimum cart value required
- Supports user-specific coupons — a code can be locked to a single buyer (e.g. a win-back offer for a lapsed customer) or open to all buyers
- Validates a coupon code at checkout in real time — checks every rule simultaneously and returns either the applicable discount or a clear reason why the code cannot be applied
- Prevents over-redemption under concurrent load — if a code has one use remaining and two buyers apply it simultaneously, exactly one succeeds
- Tracks every coupon redemption — who used it, when, and on which order — giving admins full visibility into coupon performance
- Allows admins to deactivate a coupon before its expiry date if it is being abused or was issued in error
- Provides coupon performance reporting to analytics-service: total redemptions, total discount value distributed, and revenue generated from orders using the coupon

---

## Who Uses It

- **Buyers** — apply coupon codes in the cart during checkout
- **Sellers** — create product-specific or seller-wide discount codes to drive sales
- **Admins** — create platform-wide promotional codes, manage and deactivate codes, review performance
- **cart-service** — validates and applies coupons to cart totals

---

## Key Integrations

- Called by cart-service during coupon code entry to validate and return the discount amount
- Called by order-service at purchase confirmation to record the coupon redemption
- Provides redemption data to analytics-service for promotional effectiveness reporting
