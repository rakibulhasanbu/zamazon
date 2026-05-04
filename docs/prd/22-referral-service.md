# Referral Service

**Technology:** NestJS
**Port:** 8013
**Priority:** P3 — Seller & Monetization

---

## Why NestJS

The referral and affiliate system is built around tracking events across time — a user clicks a link, days later they buy something, and the commission needs to be attributed back to the original referrer correctly. NestJS's event listener pattern handles the async attribution flow cleanly, and the application logic (commission tiers, fraud checks, attribution windows) is business-rule-heavy in a way that suits NestJS's injectable service architecture. The throughput requirement is moderate — referral events are far less frequent than orders or searches.

---

## What This Service Does

**Casual Referral Programme (any buyer)**
- Generates a unique referral link for every registered buyer automatically — no application needed
- Tracks when someone signs up using that link and credits the referrer with a store credit reward once the new buyer completes their first purchase
- Shows each buyer a simple referral dashboard: how many people they have referred and how much credit they have earned

**Formal Affiliate Programme (application required)**
- Allows any buyer to apply to become a formal affiliate — content creators, bloggers, and influencers with an audience
- Admins review and approve affiliate applications with the ability to set a custom commission rate and tier
- Issues approved affiliates a unique affiliate ID embedded in shareable product links
- Tracks link clicks with a 30-day attribution window — if someone clicks an affiliate link and purchases within 30 days, even across sessions, the commission is attributed to that affiliate
- Credits commission to the affiliate's wallet balance automatically when an attributed order is completed
- Provides a full affiliate dashboard: clicks, conversions, conversion rate, total earnings, and payout history
- Supports tiered affiliate commission rates — Bronze, Silver, and Gold affiliates earn progressively higher commission percentages based on their volume
- Detects referral fraud — same-device signups, rapid multi-account creation, and suspicious click patterns are flagged and sent to fraud-service

---

## Who Uses It

- **Buyers** — casual referral programme, share links with friends
- **Affiliates** (approved buyers) — full affiliate dashboard, commission tracking, payout management
- **Admins** — approve or reject affiliate applications, set commission rates, review fraud flags

---

## Key Integrations

- Reads affiliate attribution tags passed through order-service when a referred buyer completes a purchase
- Credits affiliate commissions to wallet-service on order completion
- Sends fraud signals to fraud-service when suspicious referral patterns are detected
- Publishes conversion events to analytics-service for affiliate programme performance reporting
