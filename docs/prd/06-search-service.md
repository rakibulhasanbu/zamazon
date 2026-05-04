# Search Service

**Technology:** Go
**Port:** 8105
**Priority:** P1 — Core Platform

---

## Why Go

Search is the highest-read-volume service on the platform. Thousands of buyers are searching simultaneously at any given moment. Go's lightweight goroutines and efficient HTTP connection pooling to Elasticsearch allow it to handle this volume without the memory cost of spinning up a thread per request, which is what Java or Python would do. The low latency matters directly to buyers — every 100ms added to search results reduces how many products buyers look at.

---

## What This Service Does

- Delivers fast, relevant product search results across the entire catalog in response to buyer queries
- Supports rich filtering so buyers can narrow results by price range, star rating, category, brand, seller, and product availability — filters can be combined freely
- Offers multiple sort options: most relevant, lowest price, highest price, newest listings, highest rated
- Provides search suggestions and autocomplete as the buyer types — showing product names, categories, and popular search terms before they finish their query
- Keeps the search index continuously updated as products are created, edited, removed, or go in and out of stock — buyers never see outdated results
- Injects sponsored product listings from ads-service into search results at designated positions, clearly marked as sponsored
- Supports related product lookups — given a product, returns other products that are similar in category, attributes, or buyer behavior patterns
- Powers the recently viewed and "customers also searched for" features on product pages

---

## Who Uses It

- **Buyers** — every search, autocomplete suggestion, and browse-by-category interaction
- **ads-service** — injects sponsored product placements into results
- **recommendation-service** — reads search behavior signals to improve recommendations

---

## Key Integrations

- Consumes `inventory.updated` events from inventory-service to reflect stock changes in search filters
- Consumes product create/update/delete events from product-service to keep the index current
- Consumes `deal.activated` and `deal.expired` events from deals-service to add and remove deal badges on search results
- Queries ads-service for sponsored placements to mix into results
