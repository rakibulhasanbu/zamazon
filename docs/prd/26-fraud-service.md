# Fraud Service

**Technology:** Go
**Port:** 8107
**Priority:** P4 — Platform Operations

---

## Why Go

Fraud detection sits directly in the payment flow — every payment attempt is scored before the charge is processed. If this service is slow, checkout is slow. Go's concurrent rule evaluation runs multiple fraud checks in parallel in microseconds without blocking the payment request. There are no garbage collection pauses that could spike latency at the worst possible moment. The service is also purely computational — it reads signals, runs rules, and emits a score — with no complex business logic that would benefit from NestJS or Django's abstractions.

---

## What This Service Does

- Scores every payment attempt in real time — before any charge is processed, the payment is evaluated against a set of fraud rules and assigned a risk score; high-risk payments are automatically blocked or flagged for manual review
- Performs velocity checks — detects abnormal patterns such as too many orders placed in a short window, multiple failed payment attempts, or an unusually high order value from a new account
- Flags suspicious IP addresses and device fingerprints — if the same device is associated with multiple different accounts, or if the login IP is from an unusual location relative to the buyer's history, the risk score increases
- Detects account takeover signals — a login from a new device, unusual location, or immediately followed by a high-value purchase triggers an elevated risk score and may require additional verification
- Detects seller fraud patterns — sellers creating fake reviews for their own products, manipulating their rating through bulk purchases and refunds, or artificially inflating their store metrics
- Detects referral and affiliate fraud — the same device or IP address being used to create multiple accounts to abuse the referral programme
- When fraud is confirmed, emits a `fraud.flagged` event that blocks the payment in payment-service and flags the order in order-service for manual review

---

## Who Uses It

- **System only** — buyers and sellers never interact with this service directly; it operates invisibly in the background on every payment and registration event

---

## Key Integrations

- Called by payment-service before processing every payment for real-time risk scoring
- Consumes `user.registered` from auth-service to score new account creation risk
- Receives referral anomaly signals from referral-service
- Publishes `fraud.flagged` events consumed by payment-service, order-service, and audit-log-service
