# Messaging Service

**Technology:** NestJS
**Port:** 8010
**Priority:** P2 — Enhanced Buyer Experience

---

## Why NestJS

Real-time chat requires a persistent WebSocket connection — the server must push new messages to the browser the moment they arrive, not wait for the browser to ask. NestJS has a built-in WebSocket gateway powered by Socket.IO that handles this without additional infrastructure. Message persistence in PostgreSQL and real-time delivery via WebSocket are both first-class in NestJS. This service handles only P2P transactional chat between buyers and sellers — AI support and admin tickets are deliberately kept in a separate service so WebSocket latency is never affected by LLM response times.

---

## What This Service Does

- Provides a direct messaging channel between a buyer and a seller, scoped to a specific order or product inquiry — a buyer can ask a seller a question about an item before purchasing, or follow up on a fulfilment issue after
- Delivers messages in real time via WebSocket — the recipient sees the message appear instantly without refreshing the page
- Persists all message threads so both parties can scroll back through the full conversation history at any time
- Shows an unread message count badge on the messaging icon in both the buyer's account and the seller's dashboard, updated in real time
- Allows file and image sharing within conversations — a buyer can share a photo of a damaged item, a seller can send a product document
- Allows admins to review flagged message threads for content that violates platform policies — sellers or buyers can flag a conversation
- Provides sellers with auto-reply templates — sellers can define standard responses to common questions (e.g. "Delivery takes 3–5 business days") so they can respond quickly without typing the same message repeatedly

---

## Who Uses It

- **Buyers** — send pre-purchase product questions and post-purchase order queries to sellers
- **Sellers** — respond to buyer messages, use auto-reply templates, manage their conversation inbox
- **Admins** — review flagged conversations and take moderation actions

---

## Key Integrations

- Links conversations to specific orders from order-service so the full order context is visible alongside the chat thread
- Links conversations to specific product listings from product-service for pre-purchase inquiries
- Notifies buyers and sellers of new messages through notification-service in real time and via email for offline recipients
