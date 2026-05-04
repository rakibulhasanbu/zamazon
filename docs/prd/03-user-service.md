# User Service

**Technology:** NestJS
**Port:** 8002
**Priority:** P1 — Core Platform

---

## Why NestJS

User profile management is rich in validation rules — address formats, phone number patterns, preference constraints, file size limits for avatars. NestJS's class-validator and TypeORM handle all of this declaratively without boilerplate. The service also reacts to events from auth-service and order-service, and NestJS's event-driven module makes this straightforward. Go would require building a validation framework from scratch; Django would work but is heavier than needed for a profile-focused service.

---

## What This Service Does

- Stores and manages each buyer's personal profile: display name, avatar image, bio, and contact preferences
- Manages the buyer's address book — they can save multiple delivery addresses, edit or remove them, and set one as the default for faster checkout
- Provides a wishlist feature where buyers can save products they are interested in but not ready to purchase yet
- Tracks which products a buyer has recently viewed so the platform can show relevant history and power recommendations
- Stores account-level settings and preferences — notification preferences, language, display settings
- Provides a read-only view of the buyer's order history by aggregating data from order-service — the buyer sees all their past orders in one place without the user-service owning that data
- Allows buyers to delete their account and all associated personal data (GDPR compliance)

---

## Who Uses It

- **Buyers** — manage their own profile, addresses, wishlist, and preferences
- **recommendation-service** — reads recently viewed history to power personalization
- **order-service** — reads default delivery address during checkout
- **admin-dashboard** — admins can view and manage user profiles (with appropriate permissions)

---

## Key Integrations

- Automatically creates a profile record when auth-service publishes the `user.registered` event
- Sends recently-viewed data to recommendation-service to improve product suggestions
- Provides address data to order-service at checkout
