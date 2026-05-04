# Seller Service

**Technology:** Django
**Port:** 8203
**Priority:** P3 — Seller & Monetization

---

## Why Django

The seller service is operations-heavy — it involves complex data views, multi-step workflows (KYC verification, listing approval, payout setup), and rich administrative interfaces. Django REST Framework accelerates building the API layer for these workflows, and Django Admin gives the internal team a powerful, free moderation interface for reviewing and approving seller accounts without building a custom admin UI. Python's data handling capabilities also make seller-facing reporting straightforward. NestJS could handle this but would require building the admin UI entirely from scratch.

---

## What This Service Does

- Manages the complete seller account lifecycle — from initial registration and profile setup through verification, active trading, and account suspension or closure
- Handles seller identity and business verification (KYC) — sellers must submit official documents (national ID, business registration, bank statement) before they can list products or receive payouts
- Routes each KYC submission to an admin review queue; an operations admin reviews the submitted documents and approves or rejects the application with a written reason
- Notifies the seller of the KYC outcome by email — approved sellers gain immediate access to list and sell; rejected sellers receive the reason and can correct and resubmit
- Maintains the seller's public-facing profile: brand name, store description, logo, banner image, seller rating, and response time metrics
- Provides sellers with a unified view of all their product listings with quick access to edit, pause, or archive listings
- Gives sellers a live view of their incoming orders with the ability to update fulfilment status — marking orders as processing, shipped (with tracking number), or delivered
- Tracks seller performance metrics over time: average rating, order volume, cancellation rate, late shipment rate, and return rate — these feed into the seller's visibility ranking
- Manages payout settings — sellers connect their bank account or payment method and configure their preferred payout schedule (weekly, biweekly, monthly)
- Enables seller-to-buyer messaging directly from the order detail view, linking to messaging-service
- Provides sellers with a summary view of their active ads campaigns managed by ads-service

---

## KYC Verification Workflow

**Seller submits documents:**
After registering, the seller uploads their required identity and business documents through the seller portal. Documents are securely stored and a verification request is created and held for admin review.

**Admin reviews:**
The operations team sees the pending submission in the admin dashboard. They can view the uploaded documents and either approve the seller (granting full access to list products and configure payouts) or reject the application with a written reason.

**Seller is notified:**
The seller receives an email with the decision. Approved sellers gain immediate access to all selling features. Rejected sellers receive the reason and can correct any issues and resubmit.

---

## Who Uses It

- **Sellers** — manage their entire store: profile, listings, orders, payouts, performance, ads
- **Admins** — approve seller registrations, review KYC submissions, suspend non-compliant accounts, manage seller categories and permissions

---

## Key Integrations

- Coordinates with media-service to securely store KYC documents uploaded by sellers
- Notifies auth-service when a seller's verification status changes so access restrictions are enforced consistently across the platform
- Surfaces the KYC review queue to the admin dashboard via admin-bff so the operations team can action submissions
- Sends KYC decision emails to sellers via notification-service
- Reads order data from order-service for the seller's order management view
- Reads earnings data from wallet-service for the seller's payout dashboard
- Reads performance metrics from analytics-service
- Links listing management to product-service for creating and editing products
