# API Gateway

**Technology:** Go
**Port:** 8000
**Priority:** P1 — Core Platform

---

## Why Go

The API Gateway sits in front of every single request the platform receives. Before any feature can respond, every call passes through here. Go was chosen because it handles tens of thousands of simultaneous connections with almost no memory overhead, and its response times are measured in microseconds — not milliseconds. Other options like Node.js or Python would add unnecessary latency to every single user interaction across the entire platform.

---

## What This Service Does

- Acts as the single entry point for all traffic from all three frontend apps (customer web, seller portal, admin dashboard)
- Verifies that every incoming request carries a valid identity token before forwarding it — unauthenticated requests are rejected immediately without touching any downstream service
- Enforces rate limits per user and per IP address to prevent abuse and protect all services from being overwhelmed
- Routes each request to the correct backend service based on the URL path
- Attaches a unique tracking ID to every request so the entire journey of that request can be traced across all services
- Handles SSL/TLS termination — all external traffic is encrypted, internal traffic uses the private network
- Validates admin permission scopes before routing requests to admin-facing services

---

## Who Uses It

- **Buyers** — every page load, search, checkout action flows through here
- **Sellers** — every dashboard action flows through here
- **Admins** — every admin operation flows through here
- Effectively transparent to all users — they never interact with it directly

---

## Key Integrations

- Sits in front of all 27 services — nothing is reachable without going through it
- Reads identity tokens issued by auth-service to validate requests
- Forwards validated requests to the appropriate downstream service
