# Notification Service

**Technology:** NestJS
**Port:** 8005
**Priority:** P1 — Core Platform

---

## Why NestJS

Notifications need to be delivered through multiple channels — email, SMS, in-app real-time alerts, and push notifications — each with its own third-party provider. NestJS has first-class support for Bull (a background job queue for scheduling and retrying sends), WebSocket gateways (for real-time in-app alerts), and adapter patterns that make plugging in SendGrid, Twilio, Firebase, and others clean and consistent. Managing all these providers in Go would require building the queue and adapter system from scratch; Django's async support is less mature for high-throughput WebSocket connections.

---

## What This Service Does

- Sends order confirmation emails to buyers immediately after a successful order is placed
- Sends shipment and delivery update notifications to buyers as their order progresses through fulfilment
- Sends payment confirmation and failure notifications so buyers always know the outcome of a charge
- Sends real-time in-app notifications through the buyer's open browser session — the notification bell updates without a page refresh
- Sends SMS messages for time-sensitive events: OTP codes for password resets, and delivery notifications for buyers who opt in
- Sends push notifications to mobile browsers for order updates and deal alerts
- Alerts sellers via email and in-app notification when their stock drops below the low-threshold they have set
- Sends welcome emails to new users when they complete registration
- Delivers promotional and marketing campaign emails — triggered by admin broadcasts or automated deal events
- Respects each user's notification preferences — every channel (email, SMS, push, in-app) can be independently opted in or out per notification category

---

## Who Uses It

- **Buyers** — receive all order and account notifications across their preferred channels
- **Sellers** — receive inventory alerts, new order alerts, and account notifications
- **Admins** — send platform-wide announcements
- All services communicate with this one via the `notification.send` Kafka topic — no service sends notifications directly

---

## Key Integrations

- Consumes `order.placed`, `order.completed`, `order.cancelled` from order-service
- Consumes `payment.confirmed`, `payment.failed` from payment-service
- Consumes `inventory.low` from inventory-service
- Consumes `user.registered` from auth-service
- Consumes `wallet.credited` from wallet-service
- Consumes `support.ticket.created`, `support.ticket.resolved` from support-service
- Consumes `notification.send` topic published by any service that needs to trigger a notification
