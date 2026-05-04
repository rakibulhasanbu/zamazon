# Product Service

**Technology:** NestJS
**Port:** 8003
**Priority:** P1 — Core Platform

---

## Why NestJS

The product catalog is the most relationship-heavy part of the system. A single product has a category, subcategory, multiple variants, dozens of attributes, a seller association, pricing rules, and media links — all interconnected. NestJS with TypeORM and MongoDB handles these rich, flexible document structures cleanly. The catalog also needs complex query building (filter by category + price range + attributes) which TypeORM's query builder handles well. Django could do this but its ORM is slower to iterate with for document-style data; Go would require writing all the query logic manually.

---

## What This Service Does

- Allows sellers to create, update, and remove product listings through the seller portal
- Organises all products into a category and subcategory tree that buyers browse — category management is controlled by admins
- Supports product variants — a single listing for a shirt can have multiple sizes (S, M, L, XL) and colors (Red, Blue, Black) each with their own stock level and price
- Stores detailed product specifications and attributes (weight, dimensions, material, compatibility) so buyers can compare products accurately
- Links products to their media files stored in media-service — main images, gallery images, and instructional videos
- Manages pricing at multiple levels: base price, sale price, bulk discount tiers, and Subscribe & Save pricing
- Tracks the status of each listing: draft (being prepared by seller), active (visible to buyers), archived (hidden but not deleted), or flagged (under review by admin)
- Hosts a Q&A section on each product page — buyers submit questions, sellers answer them, and other buyers can find answers without contacting the seller
- Syncs product updates to search-service so the search index stays current
- Applies admin-approved category and attribute rules so all listings are structured consistently

---

## Who Uses It

- **Sellers** — create and manage their product listings
- **Buyers** — browse product detail pages, read specs, submit questions
- **Admins** — moderate listings, manage categories and attributes, flag products
- **search-service** — reads product data to keep the search index up to date
- **recommendation-service** — reads product metadata to power related product suggestions

---

## Key Integrations

- Receives inventory updates from inventory-service to display accurate stock status on listings
- Notifies search-service when products are created, updated, or removed so the search index stays in sync
- Links to media-service for all product images and files
- Reacts to deal activation/expiry events from deals-service to display deal badges on listings
