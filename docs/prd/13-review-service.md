# Review Service

**Technology:** NestJS
**Port:** 8006
**Priority:** P2 — Enhanced Buyer Experience

---

## Why NestJS

Reviews have clearly defined business rules: a buyer can only review a product they have purchased, star ratings must be between 1 and 5, review text has minimum and maximum length constraints, and sellers can respond once per review. NestJS's class-validator enforces all of these rules declaratively at the input level. TypeORM cleanly manages the three-way relationship between reviews, products, and users. The moderation workflow also fits naturally into NestJS's Guards and interceptor patterns.

---

## What This Service Does

- Allows buyers to submit a review for any product they have purchased — includes a star rating (1–5), written review, and optional images
- Ensures only verified purchasers can leave reviews — the "Verified Purchase" badge is shown on qualifying reviews, giving them greater credibility
- Allows buyers to edit or delete their own reviews after submission
- Allows sellers to post one public response to each review on their products — giving context or addressing concerns
- Lets buyers mark other reviews as "helpful" — highly voted reviews are surfaced more prominently
- Calculates and maintains the aggregate star rating and total review count for each product, updated in real time after each submission
- Flags and removes reviews that violate content policies — admins can moderate reviews from the admin dashboard
- Hosts the product Q&A section — buyers ask questions publicly on a product page, sellers and other buyers can answer, and answers are visible to all future visitors
- Surfaces review summaries on product listing cards — star rating and count are shown without needing to visit the full product page

---

## Who Uses It

- **Buyers** — submit reviews, ask questions, mark reviews as helpful
- **Sellers** — respond to reviews, answer buyer questions
- **Admins** — moderate reviews and Q&A content, remove policy-violating content
- **product-service** — reads aggregate ratings to display on product listings and search results

---

## Key Integrations

- Validates purchase eligibility by checking order history with order-service before allowing a review submission
- Provides aggregate rating data to product-service for display on listings and search cards
- Sends moderation flag events to audit-log-service when content is removed
