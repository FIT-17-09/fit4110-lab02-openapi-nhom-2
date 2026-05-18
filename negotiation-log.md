# Biên bản đàm phán hợp đồng API

 Cặp đàm phán: Pair-02 (Core ↔ Analytics)
 Product: Smart Campus Alerts
 Provider: core-business-team
 Consumer: analytics-team
 Phiên: v1.0
 Ngày: 2026-05-18

---

## Issue #1

- Raised by: Consumer / Provider
- Endpoint:
- Concern:
- Proposal:
- Resolution: Accepted / Rejected / Modified
- Rationale:
- Impact:
## Issue #1 — Cursor pagination format

- Raised by: Consumer
- Endpoint: `GET /alerts`
- Concern: Consumer muốn cursor to be a safe opaque string and allow empty/null when no more pages.
- Proposal: Use `oneOf: [string, null]` for `cursor` query parameter and require `nextCursor` to be `null` when no further pages.
- Resolution: Accepted
- Rationale: `oneOf` is compatible with OpenAPI 3.1 and avoids `nullable` keyword; explicit `null` signals end-of-data.
- Impact: Provider will return `nextCursor: null` when no more pages; minimal implementation effort.

---

## Issue #2 — relatedEventId nullability

- Raised by: Provider
- Endpoint: `POST /alerts` (request payload)
- Concern: Sometimes an alert has no related event id; Consumer expected `relatedEventId` to be optional or null.
- Proposal: Use `oneOf: [string (uuid), null]` for `relatedEventId` and keep field present (may be null).
- Resolution: Accepted
- Rationale: Keeping the field avoids schema ambiguity; `null` signals absence explicitly.
- Impact: Consumers should handle `null` for `relatedEventId` in downstream processing.

---

## Issue #3 — Error responses use Problem Details

- Raised by: Consumer
- Endpoint: All error responses (4xx/5xx)
- Concern: Need a consistent error payload with machine-readable details for analytics and retry logic.
- Proposal: Use `application/problem+json` and the `Problem` schema with `errors` array for field-level issues.
- Resolution: Accepted
- Rationale: Standardized Problem Details simplifies client error handling and is required by lab rubric.
- Impact: Provider will return structured errors; Consumers update error parsers accordingly.

---

## Issue #4 — Event polymorphism (discriminator)

- Raised by: Provider
- Endpoint: `POST /events`
- Concern: Consumer needs clear discrimination between `SensorEvent` and `AccessEvent` for routing and validation.
- Proposal: Use `oneOf` with a `discriminator` on `eventType` mapping explicit values to schemas.
- Resolution: Accepted
- Rationale: Discriminator enables automatic routing and validation in runtime mock servers (e.g., Prism) and code generation.
- Impact: Consumers must include `eventType` exactly as defined (e.g., `SENSOR_READING`).

---

## Issue #5 — Location header on create

- Raised by: Consumer
- Endpoint: `POST /alerts`
- Concern: Consumer expects `Location` header with URI to newly created resource for follow-up calls.
- Proposal: Provider returns `Location` header (format: uri) and `201` with created `Alert` body.
- Resolution: Accepted
- Rationale: Conventional REST practice; aids idempotency and follow-up queries.
- Impact: Provider implements `Location` header; Consumer will use it to fetch details when needed.

---

## Issue #6 — Authentication and webhook expectations

- Raised by: Provider
- Endpoint: `webhooks.alertCreated` and secured endpoints
- Concern: Webhooks may be sent to third-party consumer endpoints; need authentication and retry semantics.
- Proposal: Use Bearer token for API endpoints; webhooks rely on consumer endpoint security (shared secret) and 200 response for acceptance. Document retry/backoff semantic out-of-band.
- Resolution: Accepted (documented)
- Rationale: Keeps API authentication consistent; webhook delivery is operational concern.
- Impact: Consumers provide secure endpoint and accept webhook payload; Provider documents retry expectations in the docs.

---

 # Chốt hợp đồng v1.0

 Provider sign-off: core-business-team / Alice Nguyen
 Consumer sign-off: analytics-team / Bob Tran
 Witness (GV/TA): TA Nguyen Van A
 Date: 2026-05-18

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
|  |  |  |
