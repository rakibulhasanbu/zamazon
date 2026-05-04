# Ads Service

**Technology:** NestJS
**Port:** 8012
**Priority:** P3 — Seller & Monetization

---

## Why NestJS

Ad serving involves moderate-throughput operations: storing campaign configurations, tracking impressions and clicks, pacing daily budgets, and injecting sponsored results. NestJS's modular architecture makes each of these concerns an injectable service. The bid management logic and budget pacing rules are business-logic-heavy, which suits NestJS's decorator and service patterns. The throughput of a portfolio-scale ad system does not require Go's raw performance; the complexity of the business logic makes NestJS the better fit.

---

## What This Service Does

- Allows sellers to create Sponsored Product campaigns — they select products to promote, set a bid amount per click, and define a daily budget cap
- Manages keyword and category targeting — sellers can bid for their product to appear when buyers search specific terms or browse specific categories
- Enforces daily budget caps — once a campaign has spent its daily limit, ads are paused automatically until the next day
- Tracks every impression (how many times an ad was shown) and every click (how many times a buyer clicked on a sponsored listing) in real time
- Injects sponsored product listings into search results and category pages at defined placements — always clearly labelled as "Sponsored" so buyers know they are promoted results
- Provides sellers with a campaign performance dashboard: impressions, clicks, click-through rate, spend to date, remaining budget, and return on ad spend
- Allows sellers to pause, resume, and end campaigns at any time
- Charges the seller's wallet balance per click — the cost-per-click is deducted from wallet-service when a sponsored result is clicked

---

## Who Uses It

- **Sellers** — create and manage ad campaigns, monitor performance, control budgets
- **Buyers** — see sponsored listings in search results and category pages (transparent to them — just clearly marked)
- **search-service** — queries this service for sponsored placements to inject into search results

---

## Key Integrations

- Provides sponsored product placements to search-service for injection into search results
- Deducts cost-per-click charges from seller wallet balance via wallet-service on each valid click
- Reports campaign spend and performance data to analytics-service
