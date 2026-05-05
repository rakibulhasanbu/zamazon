# SRS — User Service

**Technology:** NestJS
**Port:** 8002
**Database:** PostgreSQL
**Priority:** P1 — Core Platform
**PRD reference:** `docs/prd/03-user-service.md`

---

## 1. Purpose

The user-service owns all buyer profile data — personal details, address book, wishlist, recently viewed products, and account preferences. It does not handle authentication (that is auth-service) and does not own order data (that is order-service). It reacts to auth events to create profiles automatically and exposes internal gRPC endpoints so other services can read address and history data without coupling to user-service's REST API.

---

## 2. Scope

**In scope:**

- Buyer profile CRUD
- Address book management
- Wishlist management
- Recently viewed product tracking
- Account preferences
- GDPR account deletion
- Internally: gRPC endpoints for order-service and recommendation-service

**Out of scope:**

- Authentication and JWT issuance (auth-service)
- Order history data ownership (order-service)
- Avatar file storage (media-service — user-service stores the returned URL only)
- Seller profiles (seller-service)

---

## 3. Use Cases

| ID    | Actor                  | Goal                                                |
| ----- | ---------------------- | --------------------------------------------------- |
| UC-01 | System                 | Auto-create a profile when a new user registers     |
| UC-02 | Buyer                  | View and update their own profile                   |
| UC-03 | Buyer                  | Manage their address book                           |
| UC-04 | Buyer                  | Add and remove wishlist items                       |
| UC-05 | Buyer                  | View recently viewed products                       |
| UC-06 | Buyer                  | Update notification and display preferences         |
| UC-07 | Buyer                  | Delete their account and all personal data          |
| UC-08 | order-service          | Read a buyer's default delivery address at checkout |
| UC-09 | recommendation-service | Read a buyer's recently viewed product history      |
| UC-10 | admin-bff              | View and manage a buyer's profile                   |

---

## 4. Functional Requirements

### 4.1 Profile Management

- **FR-01** — On receiving a `user.registered` Kafka event, the service MUST create a `user_profile` record using the `user_id` from the event. The profile is created with empty optional fields; no user action is required.
- **FR-02** — A buyer can retrieve their own profile. The response includes `display_name`, `avatar_url`, `bio`, `email` (read-only, from the JWT), `created_at`.
- **FR-03** — A buyer can update `display_name`, `avatar_url`, and `bio`. `email` is not editable here — that is auth-service's responsibility.
- **FR-04** — `avatar_url` must be a URL returned by media-service. The user-service does not accept raw file uploads; it stores the URL only.
- **FR-05** — `display_name` must be 2–50 characters. `bio` must be 0–300 characters.

### 4.2 Address Book

- **FR-06** — A buyer can add a delivery address. Required fields: `full_name`, `line1`, `city`, `postal_code`, `country`, `phone`. Optional: `line2`, `label` (home / work / other).
- **FR-07** — A buyer can have at most **20 addresses**. Attempting to add a 21st returns a 422 error.
- **FR-08** — A buyer can update any field of an existing address.
- **FR-09** — A buyer can delete an address. If the deleted address was the default, the service sets no default (the buyer must choose a new one before checkout).
- **FR-10** — A buyer can mark any address as the default. Only one address can be the default at a time; marking a new default unsets the previous one atomically.
- **FR-11** — `phone` must match E.164 format. `postal_code` is validated against the selected `country` pattern. `country` must be a valid ISO 3166-1 alpha-2 code.

### 4.3 Wishlist

- **FR-12** — A buyer can add a product to their wishlist by `product_id`. Duplicate entries are silently ignored (idempotent).
- **FR-13** — A buyer can remove a product from their wishlist by `product_id`.
- **FR-14** — A buyer can view their full wishlist ordered by `added_at` descending. The response includes `product_id` and `added_at` only — product details are fetched by the frontend from product-service directly.
- **FR-15** — Wishlist is capped at **500 items**. Adding beyond the cap returns a 422 error.

### 4.4 Recently Viewed

- **FR-16** — When a buyer views a product, the frontend calls `POST /users/me/recently-viewed` with a `product_id`. The service upserts the record — if the product was already viewed, it updates `viewed_at` to now.
- **FR-17** — The recently viewed list is capped at **50 products** per user (ring buffer — oldest entry is dropped when the cap is exceeded).
- **FR-18** — A buyer can retrieve their recently viewed list ordered by `viewed_at` descending.

### 4.5 Account Preferences

- **FR-19** — A buyer can update their notification preferences: `email_notifications` (bool), `sms_notifications` (bool), `push_notifications` (bool).
- **FR-20** — A buyer can set `language` (BCP 47 tag, e.g. `en-US`) and `currency` (ISO 4217 code, e.g. `USD`).
- **FR-21** — Preferences are created with defaults on profile creation: all notifications enabled, `language = en-US`, `currency = USD`.

### 4.6 Account Deletion (GDPR)

- **FR-22** — A buyer can request account deletion. The service immediately soft-deletes the profile (`deleted_at = now`) and returns 200.
- **FR-23** — A scheduled job runs daily and hard-deletes all profiles where `deleted_at` is older than **30 days**, removing all associated addresses, wishlist items, recently viewed entries, and preferences.
- **FR-24** — After soft-deletion, all REST endpoints for the deleted user return 404. The profile is invisible to all other services.
- **FR-25** — The service publishes a `user.deleted` Kafka event on soft-delete so downstream services (recommendation-service, notification-service) can clean up their own data.

---

## 5. Non-Functional Requirements

| ID     | Requirement                                                                                  |
| ------ | -------------------------------------------------------------------------------------------- |
| NFR-01 | Profile read (GET /users/me) must respond in < 100 ms at p99                                 |
| NFR-02 | The service must handle 500 concurrent requests without degradation                          |
| NFR-03 | All endpoints require a valid JWT — no public endpoints                                      |
| NFR-04 | Buyers can only access their own data — a buyer cannot read another buyer's profile via REST |
| NFR-05 | admin-bff can read any profile using a service-level JWT with `admin` role                   |
| NFR-06 | All personal data fields (`display_name`, `bio`, addresses) are stored encrypted at rest     |
| NFR-07 | The service must be stateless — no in-process session state; all state lives in PostgreSQL   |

---

## 6. Database Schema

```sql
-- Core profile record, one per registered user
CREATE TABLE user_profiles (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  auth_user_id UUID NOT NULL UNIQUE,     -- matches sub from auth-service JWT
  display_name VARCHAR(50),
  avatar_url   TEXT,
  bio          VARCHAR(300),
  created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  deleted_at   TIMESTAMPTZ                           -- soft delete
);

CREATE TABLE addresses (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  label       VARCHAR(20) DEFAULT 'other',          -- home | work | other
  full_name   VARCHAR(100) NOT NULL,
  line1       VARCHAR(200) NOT NULL,
  line2       VARCHAR(200),
  city        VARCHAR(100) NOT NULL,
  state       VARCHAR(100),
  postal_code VARCHAR(20)  NOT NULL,
  country     CHAR(2)      NOT NULL,                -- ISO 3166-1 alpha-2
  phone       VARCHAR(20)  NOT NULL,                -- E.164
  is_default  BOOLEAN      NOT NULL DEFAULT false,
  created_at  TIMESTAMPTZ  NOT NULL DEFAULT now(),
  updated_at  TIMESTAMPTZ  NOT NULL DEFAULT now()
);

-- Partial unique index: only one default address per user
CREATE UNIQUE INDEX one_default_address_per_user
  ON addresses (user_id)
  WHERE is_default = true;

CREATE TABLE wishlist_items (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  product_id UUID NOT NULL,
  added_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, product_id)
);

CREATE TABLE recently_viewed (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  product_id UUID NOT NULL,
  viewed_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, product_id)
);

CREATE TABLE user_preferences (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID NOT NULL UNIQUE REFERENCES user_profiles(id) ON DELETE CASCADE,
  email_notifications BOOLEAN     NOT NULL DEFAULT true,
  sms_notifications   BOOLEAN     NOT NULL DEFAULT true,
  push_notifications  BOOLEAN     NOT NULL DEFAULT true,
  language            VARCHAR(10) NOT NULL DEFAULT 'en-US',  -- BCP 47
  currency            CHAR(3)     NOT NULL DEFAULT 'USD',     -- ISO 4217
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 7. REST API Contract

All routes require `Authorization: Bearer <JWT>`. The JWT `sub` field is used to identify the caller.

### Profile

| Method | Path         | Description                          | Auth       |
| ------ | ------------ | ------------------------------------ | ---------- |
| GET    | `/users/me`  | Get own profile                      | Buyer      |
| PATCH  | `/users/me`  | Update display_name, avatar_url, bio | Buyer      |
| DELETE | `/users/me`  | Soft-delete own account              | Buyer      |
| GET    | `/users/:id` | Get any profile                      | Admin only |
| PATCH  | `/users/:id` | Update any profile                   | Admin only |

**GET /users/me — response 200:**

```json
{
  "id": "uuid",
  "display_name": "Jane Doe",
  "avatar_url": "https://cdn.zamazon.com/avatars/xyz.jpg",
  "bio": "Avid shopper.",
  "email": "jane@example.com",
  "created_at": "2025-01-10T09:00:00Z"
}
```

**PATCH /users/me — request body:**

```json
{
  "display_name": "Jane D.",
  "avatar_url": "https://cdn.zamazon.com/avatars/new.jpg",
  "bio": "Updated bio."
}
```

### Addresses

| Method | Path                              | Description        |
| ------ | --------------------------------- | ------------------ |
| GET    | `/users/me/addresses`             | List all addresses |
| POST   | `/users/me/addresses`             | Add new address    |
| PATCH  | `/users/me/addresses/:id`         | Update address     |
| DELETE | `/users/me/addresses/:id`         | Remove address     |
| PATCH  | `/users/me/addresses/:id/default` | Set as default     |

**POST /users/me/addresses — request body:**

```json
{
  "label": "home",
  "full_name": "Jane Doe",
  "line1": "123 Main St",
  "line2": "Apt 4B",
  "city": "New York",
  "state": "NY",
  "postal_code": "10001",
  "country": "US",
  "phone": "+12125551234"
}
```

### Wishlist

| Method | Path                             | Description                       |
| ------ | -------------------------------- | --------------------------------- |
| GET    | `/users/me/wishlist`             | Get wishlist (paginated, 20/page) |
| POST   | `/users/me/wishlist`             | Add product to wishlist           |
| DELETE | `/users/me/wishlist/:product_id` | Remove from wishlist              |

**POST /users/me/wishlist — request body:**

```json
{ "product_id": "uuid" }
```

### Recently Viewed

| Method | Path                        | Description                  |
| ------ | --------------------------- | ---------------------------- |
| GET    | `/users/me/recently-viewed` | Get recently viewed (max 50) |
| POST   | `/users/me/recently-viewed` | Record a product view        |

**POST /users/me/recently-viewed — request body:**

```json
{ "product_id": "uuid" }
```

### Preferences

| Method | Path                    | Description        |
| ------ | ----------------------- | ------------------ |
| GET    | `/users/me/preferences` | Get preferences    |
| PATCH  | `/users/me/preferences` | Update preferences |

**PATCH /users/me/preferences — request body:**

```json
{
  "email_notifications": true,
  "sms_notifications": false,
  "push_notifications": true,
  "language": "en-US",
  "currency": "USD"
}
```

---

## 8. gRPC Internal API

These endpoints are not exposed through api-gateway. They are called service-to-service on the internal Kubernetes network only.

```protobuf
service UserService {
  // Called by order-service at checkout to get the buyer's default address
  rpc GetDefaultAddress (GetDefaultAddressRequest) returns (AddressResponse);

  // Called by recommendation-service to build the personalization model
  rpc GetRecentlyViewed (GetRecentlyViewedRequest) returns (RecentlyViewedResponse);

  // Called by admin-bff to fetch a profile for the admin dashboard
  rpc GetProfile (GetProfileRequest) returns (ProfileResponse);
}

message GetDefaultAddressRequest {
  string user_id = 1;
}

message AddressResponse {
  string id          = 1;
  string full_name   = 2;
  string line1       = 3;
  string line2       = 4;
  string city        = 5;
  string state       = 6;
  string postal_code = 7;
  string country     = 8;
  string phone       = 9;
}

message GetRecentlyViewedRequest {
  string user_id = 1;
  int32  limit   = 2;  // max 50
}

message RecentlyViewedResponse {
  repeated RecentlyViewedItem items = 1;
}

message RecentlyViewedItem {
  string product_id = 1;
  string viewed_at  = 2;  // ISO 8601
}
```

---

## 9. Kafka Events

### Consumed

| Topic             | Action                                                           |
| ----------------- | ---------------------------------------------------------------- |
| `user.registered` | Create `user_profiles` + `user_preferences` record with defaults |

**Expected event payload:**

```json
{
  "event": "user.registered",
  "user_id": "uuid",
  "email": "jane@example.com",
  "role": "buyer",
  "registered_at": "2025-01-10T09:00:00Z"
}
```

### Produced

| Topic          | Trigger                                  |
| -------------- | ---------------------------------------- |
| `user.deleted` | Buyer soft-deletes their account (FR-25) |

**Emitted payload:**

```json
{
  "event": "user.deleted",
  "user_id": "uuid",
  "deleted_at": "2025-06-01T12:00:00Z"
}
```

---

## 10. Error Responses

All errors follow a consistent shape:

```json
{
  "statusCode": 422,
  "error": "Unprocessable Entity",
  "message": "Address limit reached. Maximum 20 addresses per account."
}
```

| Scenario                                       | HTTP Status |
| ---------------------------------------------- | ----------- |
| JWT missing or invalid                         | 401         |
| Buyer accessing another buyer's data           | 403         |
| Profile / address / item not found             | 404         |
| Validation failure (bad format, missing field) | 422         |
| Address cap (20) exceeded                      | 422         |
| Wishlist cap (500) exceeded                    | 422         |

---

## 11. Inter-Service Dependencies

| Service                | How                                | When                                                        |
| ---------------------- | ---------------------------------- | ----------------------------------------------------------- |
| auth-service           | Kafka consumer (`user.registered`) | Profile creation                                            |
| media-service          | HTTP (via api-gateway)             | Buyer uploads avatar — user-service stores the returned URL |
| order-service          | gRPC consumer (calls user-service) | Checkout — reads default address                            |
| recommendation-service | gRPC consumer (calls user-service) | Reads recently viewed for model input                       |
| notification-service   | Kafka producer (`user.deleted`)    | Cleans up notification preferences                          |
| admin-bff              | gRPC consumer (calls user-service) | Admin dashboard profile views                               |
