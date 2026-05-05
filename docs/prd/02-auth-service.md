# Auth Service

**Technology:** NestJS
**Port:** 8001
**Priority:** P1 — Core Platform

---

## Why NestJS

Authentication is about complex, layered logic — OAuth handshakes, token refresh flows, password reset journeys, role assignments. NestJS has a mature, battle-tested authentication ecosystem (Passport.js, Guards, Strategies) that handles all of these patterns cleanly out of the box. Writing the same flows in Go would require building those patterns from scratch.

---

## What This Service Does

- Allows new users to create an account using their email and password
- Allows existing users to log in and log out securely
- Supports social login through Google so users do not need to create a separate password
- Issues short-lived access tokens and long-lived refresh tokens — access tokens expire quickly for security, refresh tokens allow seamless re-authentication without asking the user to log in again
- Sends a one-time password to the user's email when they request a password reset
- Verifies new accounts via an email confirmation link before full access is granted
- Assigns roles to users: buyer, seller, or admin — each role unlocks different parts of the platform
- For admin users, issues a token that carries specific permission scopes (e.g. `analytics:read`, `users:rw`) so different admin roles have precisely scoped access
- Tracks whether a seller account has been verified — an unverified seller cannot list products or receive payouts; the verification status is kept in sync automatically whenever seller-service completes a KYC review
- Revokes tokens immediately on logout or when a security event is detected

---

## Who Uses It

- **Buyers** — register, log in, reset password
- **Sellers** — register, log in; the account tracks their verification status which gates access to selling features
- **Admins** — log in, receive scoped permission tokens
- **api-gateway** — validates tokens issued by this service on every request

---

## Key Integrations

- Publishes a `user.registered` event when a new account is created — picked up by user-service (to create the profile) and notification-service (to send a welcome email)
- Coordinates with user-service to ensure the profile record is created alongside the auth record
- Receives seller verification decisions from seller-service and enforces the resulting access restrictions across the platform — an unverified seller is blocked from listing products or receiving payouts regardless of which part of the platform they access
- All other services trust tokens issued by this service
