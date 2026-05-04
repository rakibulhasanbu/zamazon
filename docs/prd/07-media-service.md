# Media Service

**Technology:** Go
**Port:** 8106
**Priority:** P1 — Core Platform

---

## Why Go

Uploading product images means streaming large files from the browser directly to storage without loading the entire file into memory first. Go handles this with streaming I/O and passes data straight to MinIO in chunks — no garbage collection pauses mid-stream, no memory spikes. Python and Node.js buffer files differently and can cause memory pressure under concurrent large uploads, which matters when hundreds of sellers are uploading product images at the same time.

---

## What This Service Does

- Accepts file uploads from sellers (product images, banners, listing videos) and from buyers (profile avatars, review images)
- Validates every uploaded file before storing it — checks file type (only accepted image and video formats), file size limits, and basic content safety
- Stores all files in MinIO, the platform's S3-compatible object storage, where they are durably persisted
- Generates a public CDN URL for every stored file so it can be served at high speed to buyers worldwide through Cloudflare's edge network — product images load fast regardless of the buyer's location
- Automatically creates multiple thumbnail sizes for product images (small for search results, medium for listing cards, large for the product detail page) so the right size is always served
- Handles deletion — when a product is removed or a seller account is closed, associated media files are cleaned up from storage
- Tracks which files belong to which product, user, or seller so ownership is always clear

---

## Who Uses It

- **Sellers** — upload product images, banners, and brand assets through the seller portal
- **Buyers** — upload profile avatars and images attached to reviews
- **product-service** — links to files stored here for all product image displays
- **review-service** — links to buyer-uploaded review images stored here

---

## Key Integrations

- Files are referenced by URL in product-service, user-service, and review-service — those services never store the files themselves
- All public file URLs are served through Cloudflare CDN, not directly from MinIO
- Coordinates with product-service and user-service to clean up files when records are deleted
