# Lleno One — Product & API Contract v0.2.1

Status: FROZEN FOR MVP IMPLEMENTATION  
Market: Mexico  
Language: Spanish  
Currency: MXN  
Product domain: lleno.io  
Web app: app.lleno.io  
API: api.lleno.io

---

# 1. Purpose

This document is the canonical product and integration contract for the first production version of Lleno.

Both `lleno-app` and `lleno-api` MUST implement against this contract.

Neither implementation may invent incompatible endpoints, fields, states, business rules, limits, or workflows.

If implementation reveals a necessary contract change:

1. update this contract first
2. update OpenAPI/schema fixtures
3. increment contract version
4. only then modify app/API implementations

The contract is more authoritative than either codebase.

---

# 2. Product Definition

Lleno converts everyday consumer receipts into personal consumer intelligence.

The first product is:

# Despensa Wrapped

A user uploads receipts from purchases made in Mexico.

Lleno:

1. reads the receipts
2. removes unnecessary sensitive information
3. extracts purchases
4. normalizes products
5. calculates deterministic spending statistics
6. discovers patterns
7. generates playful consumer-personality content
8. produces a visual, interactive Wrapped
9. offers account-level Lleno One subscription for ongoing personal history and reports

Lleno is NOT an expense tracker.

Lleno is:

> Consumer intelligence for ordinary people.

---

# 3. Core MVP Funnel

```text
lleno.io
    ↓
app.lleno.io
    ↓
anonymous session
    ↓
create analysis
    ↓
use five monthly Free receipts
    ↓
process
    ↓
light review if necessary
    ↓
FREE WRAPPED
    ↓
share
    ↓
upgrade to One at MXN $49/month
    ↓
use 100 monthly receipts and owned purchase history
```

Primary business metric:

`free_wrapped_to_paid_conversion`

Primary organic-growth metric:

`wrapped_share_rate`

Supporting funnel:

```text
landing
→ app
→ upload started
→ successful processing
→ review completed
→ wrapped viewed
→ share
→ checkout viewed
→ checkout started
→ payment completed
```

---

# 4. User Model

The first analysis MUST NOT require:

- email
- password
- name
- account creation form

The platform should establish an anonymous authenticated identity automatically.

Recommended mechanism:

Supabase Anonymous Auth.

Internally:

```text
anonymous auth user
        ↓
profile
        ↓
household
        ↓
household_member
```

Every user belongs to a household from day one.

MVP households contain one user.

The database MUST support multiple household members later without schema redesign.

Account conversion can happen later by linking the anonymous identity to a permanent authentication method.

---

# 5. Free Tier

Free users may create multiple free analyses.

Do NOT implement a lifetime "one free analysis per person" restriction in MVP.

Reasonable abuse protection and rate limits are allowed.

Each analysis/job accepts at most 15 receipt documents, subject to the owner's
remaining monthly account allowance (5 on Free, 100 on One).

Supported formats:

- JPG
- JPEG
- PNG
- PDF

Receipts may come from any consumer purchase.

Examples include:

- supermarkets
- convenience stores
- pharmacies
- warehouse clubs
- local stores
- pet stores
- household goods
- food-related businesses
- other everyday consumer merchants

Do not restrict the parser architecture to known retailers.

---

# 6. Free Report

The free Wrapped MUST provide real value.

Do not intentionally make the free experience useless.

Free report includes:

## Summary

- total amount spent
- number of receipts
- number of products/items analyzed
- detected date range
- stores detected

## Categories

- category totals
- category percentages
- top category

## Personality

A creative, funny, data-grounded "Despensa Personality".

This is a CORE acquisition feature.

## Basic savings insight

At least one simple savings or price observation when supported by the user's own data.

Examples:

- same normalized product purchased at different prices
- same category showing meaningful price variation
- different versions/brands of similar products purchased at different normalized unit prices

Never fabricate savings.

## Share Card

A social card generated from sanitized report data.

Primary formats should work for:

- WhatsApp Status
- Instagram Stories
- TikTok/Reels screenshots
- Facebook

Default sharing MUST NOT expose:

- exact total spending
- payment details
- personal identifiers
- loyalty identifiers
- addresses
- receipt numbers
- card information

---

# 7. Despensa Personality Rules

Personality content should be:

- Mexican Spanish
- playful
- memorable
- chusco
- friendly
- concise
- data-grounded
- suitable for public sharing

It may use light cultural humor.

It must NOT infer or mock:

- income level
- poverty
- health diagnoses
- body weight
- addiction
- religion
- political beliefs
- ethnicity
- sexual orientation
- pregnancy
- medical conditions
- relationship status
- other sensitive traits

AI receives structured statistics rather than raw receipts whenever possible.

Personality response contract:

```json
{
  "id": "personality_xxx",
  "emoji": "🥩",
  "title": "El Norteño Constitucional",
  "one_liner": "La carne no es una categoría. Es infraestructura.",
  "description": "Tu categoría dominante fue proteína y apareció en varios de tus tickets.",
  "evidence": [
    {
      "metric": "category_share",
      "label": "Proteína",
      "value": 0.31
    }
  ],
  "share_text": "Al parecer para mí la carne no es comida. Es infraestructura."
}
```

All evidence values must come from deterministic analytics.

---

# 8. Complete Analysis

Lleno One costs exactly MXN $49.00 per provider monthly billing cycle.
The API chooses the One plan and price; the client cannot set either. A verified
provider payment grants access only through its verified paid-through timestamp.
Authorization, checkout redirects, and browser return parameters never grant
access. Cancellation stops renewal and leaves access through the already paid
period. Failed renewal grants no additional grace period. Account cancellation
does not delete sanitized history or reports already earned.

Free users may process five successful unique receipt documents per owner-local
calendar month. One users may process 100. A receipt document counts once when
sanitized extraction is durably accepted, including when item review is needed.
Retries of the same receipt record cannot consume capacity twice. A document
rejected before accepted extraction or a terminal processing failure releases
its reservation. Exact same-file uploads by the same owner in the same month are
duplicates and do not consume another allowance slot. The allowance is atomic
across concurrent analyses; usage reports used, reserved, remaining and reset
time. The 100 limit is an initial beta ceiling and must not be advertised until
measured cost acceptance.

An analysis accepts at most 15 receipt documents. This is an independent job
bound; monthly allowance cannot be bypassed by creating more analyses. Each
document represents one receipt. Multi-receipt documents must be split by the
client; the API does not silently truncate or guess accounting units.

Supported JPEG, PNG and PDF files are at most 10 MiB. PDFs are limited to five
pages; images are limited to 20 million decoded pixels per page and 12,000 pixels
per side. Signed upload URLs expire after exactly 600 seconds. Oversized or
out-of-bound documents are rejected before OCR.

---

# 9. Full Report

Everything from Free plus:

## Product recurrence

Identify products purchased repeatedly.

## Product normalization

Recognize retailer abbreviations and aliases as canonical products when confidence is sufficient.

Example:

```text
COCA 2.5 NR
COCACOLA 2500
CC PET 2.5
```

may normalize to:

```text
Coca-Cola Original
2.5 L
```

## Unit normalization

Where possible normalize:

- ml → L
- g → kg
- unit count
- packs
- price per unit
- price per liter
- price per kilogram

## Price history

Compare prices observed in the user's own receipts.

No external retailer price lookup in MVP.

## Store comparison

Compare observed prices only when data supports a meaningful comparison.

## Detailed savings opportunities

Example:

> If recurring products had been purchased at the lowest normalized unit price you personally observed during this period, the modeled difference would have been approximately $X.

Language must clearly indicate this is historical/modelled, not guaranteed savings.

## Patterns

Examples:

- recurring products
- category concentration
- store concentration
- average ticket
- higher/lower spending contexts supported by dates
- repeated price differences
- product alternatives already purchased by the user

## Deeper insight cards

AI may describe patterns, but numbers come from deterministic computation.

---

# 10. Explicitly Out of MVP

Do NOT implement:

- automatic recurring purchases
- Amazon purchasing
- Mercado Libre purchasing
- retailer APIs
- live retailer prices
- external affiliate links
- household invitations
- shared household editing
- native iOS/Android app
- PDF reports
- grocery-list management
- budgets
- price alerts
- automatic reorder schedules
- external product recommendations
- guaranteed savings
- market-wide inflation calculations

The data model MAY anticipate these capabilities.

The product MUST NOT expose unfinished functionality as if it worked.

---

# 11. Recommended Technical Boundaries

## lleno.io

Public marketing/editorial website.

Separate existing project.

Responsibilities:

- SEO
- articles
- acquisition
- product explanation
- CTA to app.lleno.io

Not part of this implementation contract beyond URLs.

## app.lleno.io

Frontend product.

Recommended:

- React
- TypeScript
- Vite
- Tailwind
- TanStack Query
- React Router
- Zod
- React Hook Form
- Motion where useful

No direct database access for product data.

## api.lleno.io

Application API/orchestrator.

Recommended:

- TypeScript
- Cloudflare Workers
- Hono
- OpenAPI
- Supabase PostgreSQL/Auth
- Cloudflare R2
- Cloudflare Queues

## receipt processor

Background processing service.

Recommended:

- Python
- Docker
- OpenCV
- OCR engine abstraction
- Cloudflare Containers

---

# 12. Storage Rules

Original receipt files live in private object storage.

Recommended:

Cloudflare R2.

Original receipt files MUST:

- never be public
- use unguessable object keys
- be accessible only through short-lived signed access when required
- be deleted as soon as successful processing makes them unnecessary
- always have a hard expiry no later than 5 hours after R2 object creation

The API uses the R2 `object_created_at + 5 hours` deadline. Upload confirmation
cannot extend it. Application cleanup must remove unconfirmed and orphaned
objects with operational margin; asynchronous R2 lifecycle deletion is only a
coarse backstop, not proof of a five-hour deadline. The receipt row remains
after raw-object deletion.

Target:

```text
successful receipt
→ delete original shortly after successful extraction

failed/stalled/unconfirmed receipt
→ force delete at R2 object_created_at + 5h
```

The five-hour limit is a maximum retention period, not the normal desired retention period.

---

# 13. Sensitive Information

Receipts may contain:

- card suffixes
- authorization codes
- loyalty IDs
- names
- customer IDs
- addresses
- transaction IDs
- payment references

Before durable persistence, the processing pipeline must remove fields not necessary for Lleno analytics.

Do not persist complete raw OCR output by default.

Do not log:

- raw OCR
- full receipt image URLs
- payment data
- loyalty identifiers

Store only data necessary for the product.

---

# 14. Receipt Processing Pipeline

Canonical flow:

```text
uploaded receipt
      ↓
image/PDF preparation
      ↓
preprocessing
      ↓
primary OCR
      ↓
receipt parser
      ↓
confidence evaluation
      ↓
fallback vision/OCR if needed
      ↓
PII filtering
      ↓
structured receipt
      ↓
product normalization
      ↓
confidence evaluation
      ↓
user review if necessary
      ↓
deterministic analytics
      ↓
AI narrative/personality
      ↓
report
```

---

# 15. AI Boundaries

AI MUST NOT be the source of truth for arithmetic.

AI MUST NOT calculate authoritative:

- totals
- category amounts
- percentages
- savings
- price differences
- receipt counts
- product counts
- unit prices
- date ranges

These are produced deterministically.

AI MAY assist with:

- OCR fallback
- ambiguous product parsing
- product normalization
- category resolution
- alias matching
- insight wording
- personality generation
- humorous copy

All structured AI outputs should be schema validated.

---

# 16. OCR Strategy

Use a hybrid pipeline.

Primary path should be inexpensive/open-source.

Candidate engines:

- PaddleOCR
- Tesseract

The processor must expose OCR through an interface so engine choice is replaceable.

Example:

```python
class OCREngine:
    def extract(self, document) -> OCRResult:
        ...
```

Do not hardwire product domain logic to one OCR provider.

Fallback should occur based on confidence or parse failure.

The system should eventually benchmark OCR performance against Mexican receipts.

---

# 17. Product Normalization Pipeline

Recommended hierarchy:

```text
raw sanitized label
      ↓
exact known alias
      ↓
normalized string comparison
      ↓
heuristic/fuzzy candidate search
      ↓
existing product candidate
      ↓
AI disambiguation if needed
      ↓
new canonical product OR unresolved
```

Never silently merge low-confidence products.

Maintain:

- canonical product
- aliases
- normalization confidence
- normalization source
- review status

Human corrections should become useful alias evidence later.

---

# 18. Core Database Entities

Required conceptual entities:

```text
profiles

households
household_members

analyses

receipts
receipt_items

stores

products
product_aliases
brands
categories

processing_jobs
review_issues

reports
insights
personalities

payments
entitlements

analytics_events
```

---

# 19. Analysis States

Canonical states:

```text
created
uploading
ready_to_process
queued
processing
needs_review
generating_report
completed
failed
expired
```

An analysis may transition:

```text
created
→ uploading
→ ready_to_process
→ queued
→ processing
→ needs_review
→ generating_report
→ completed
```

or:

```text
processing
→ generating_report
→ completed
```

Failure:

```text
queued/processing/generating_report
→ failed
```

Before processing begins, a valid upload-slot request may transition
`ready_to_process → uploading` when receipt capacity remains. Upload-slot
issuance may also keep an analysis in `uploading` when it is already there.
Completion moves `uploading → ready_to_process` only when at least one receipt
is confirmed uploaded and no active pending upload slot remains. An active
pending slot has `status=pending_upload` and `upload_expires_at > now()`.
Expired pending slots become `failed` with an internal failure reason and do
not consume capacity.

Paid analysis may transition from `completed` back to:

```text
uploading
→ ready_to_process
→ ...
```

when additional receipts are added.

---

# 20. Receipt States

```text
pending_upload
uploaded
queued
processing
needs_review
processed
failed
deleted
```

Deletion of the raw object does not mean deleting extracted structured purchase data.

---

# 21. Review Experience

Review should be lightweight.

The frontend should normally show only ambiguous items.

Example:

> Encontramos 63 productos.  
> 3 necesitan tu ayuda.

Each review issue may include:

- receipt
- original sanitized line
- interpreted product
- quantity
- price
- suggested correction
- confidence
- persisted resolution (`null` while unresolved)

User can:

- accept
- edit
- mark unrecognized
- remove line

`GET /analyses/{analysis_id}/review` returns every ambiguous issue while
review is open, including issues that already have a persisted resolution.
Refresh and reopen restore successful prior actions from this resource.
Unsaved form drafts are frontend-only and are not canonical persistence.

Until review confirmation, the user may change an issue's action or
correction. After confirmation, analysis continues on the canonical
lifecycle; further review mutations are invalid.

---

# 22. API Authentication

App establishes anonymous Supabase authentication.

API receives:

```http
Authorization: Bearer <supabase-jwt>
```

API validates token.

API derives user identity from token.

Clients MUST NOT submit arbitrary `user_id` ownership fields.

Authorization must verify household/analysis ownership server-side.

---

# 23. API Base

```text
https://api.lleno.io/v1
```

---

# 24. Core API Endpoints

## Health

```text
GET /health
```

Response:

```json
{
  "status": "ok",
  "version": "..."
}
```

---

## Current user

```text
GET /me
```

Response includes:

```json
{
  "profile_id": "...",
  "anonymous": true,
  "household_id": "...",
  "verified": false,
  "plan": "free",
  "subscription": null,
  "timezone": "America/Matamoros",
  "usage": {
    "month": "2026-10",
    "allowance": 5,
    "used": 0,
    "reserved": 0,
    "remaining": 5,
    "resets_at": "2026-11-01T06:00:00Z",
    "timezone": "America/Matamoros"
  }
}
```

`verified` is derived from the validated Supabase identity and contains no
email address. `usage` is atomically reserved across the account's concurrent
analyses. See the OpenAPI `Me` schema for exact fields.

---

## Create analysis

```text
POST /analyses
```

Request:

```json
{
  "mode": "free"
}
```

Response:

```json
{
  "id": "ana_xxx",
  "mode": "free",
  "status": "created",
  "max_receipts": 15,
  "receipt_count": 0,
  "created_at": "..."
}
```

---

## List history

```text
GET /analyses
```

Return user's analyses ordered newest first.

---

## Get analysis

```text
GET /analyses/{analysis_id}
```

Example:

```json
{
  "id": "ana_xxx",
  "mode": "free",
  "status": "processing",
  "max_receipts": 15,
  "receipt_count": 2,
  "progress": {
    "stage": "normalizing_products",
    "percent": 72,
    "message": "Organizando tus compras"
  }
}
```

---

## Request upload slots

```text
POST /analyses/{analysis_id}/uploads
```

Request:

```json
{
  "files": [
    {
      "filename": "ticket.jpg",
      "content_type": "image/jpeg",
      "size_bytes": 2849442
    }
  ]
}
```

Response:

```json
{
  "uploads": [
    {
      "receipt_id": "rec_xxx",
      "upload_url": "<signed-url>",
      "expires_at": "...",
      "required_headers": {}
    }
  ]
}
```

API validates:

- caller ownership, the 15-document analysis ceiling, and the account's atomic monthly usage
- MIME type
- allowed size: 10485760 bytes (10 MiB) per file

Signed PUT URLs expire after 600 seconds. PDFs are limited to five pages;
decoded images are limited to 20 million pixels per page and 12,000 pixels per
side. The browser uploads directly to private R2; the API never proxies receipt
bytes.

---

## Confirm uploads

```text
POST /analyses/{analysis_id}/uploads/complete
```

Request:

```json
{
  "receipt_ids": ["rec_xxx"]
}
```

The API validates object existence. Any non-empty subset of issued receipt IDs
may be confirmed. Repeating confirmation of an already uploaded receipt is
idempotent and does not change its original `uploaded_at` or `delete_after`.

---

## Start processing

```text
POST /analyses/{analysis_id}/process
```

Must be idempotent.

When the analysis is `ready_to_process`, the first valid request creates one
processing run and returns `200` with `status: "queued"`. Repeating the request
while that run is `queued` MUST return the same `200` queued response without
creating another run. Once a processor has advanced the analysis to
`processing`, or the analysis has entered a later processing/report state, a
new request MUST return `409 PROCESSING_ALREADY_STARTED` and MUST NOT restart
the run.

Returns:

```json
{
  "analysis_id": "ana_xxx",
  "status": "queued"
}
```

---

## Get review

```text
GET /analyses/{analysis_id}/review
```

Valid while analysis `status` is `needs_review`.

Returns every ambiguous review issue for the open review, including issues
that already have a persisted resolution. It does not filter to unresolved
issues only. Each issue exposes `resolution`, which is `null` while
unresolved and otherwise the latest valid persisted action.

Response:

```json
{
  "analysis_id": "ana_xxx",
  "total_items": 63,
  "issues": [
    {
      "id": "review_xxx",
      "receipt_id": "rec_xxx",
      "item_id": "item_xxx",
      "raw_label": "CC PET 2.5",
      "suggested_name": "Coca-Cola Original 2.5 L",
      "quantity": 1,
      "line_total": 44.0,
      "confidence": 0.67,
      "resolution": null
    }
  ]
}
```

---

## Update review item

```text
PATCH /analyses/{analysis_id}/review/{issue_id}
```

Valid while analysis `status` is `needs_review`. Updates that issue's
persisted resolution and returns the full updated canonical issue.

PATCH is a resource update: repeating the same valid request MUST NOT
duplicate side effects. The latest valid persisted resolution is
authoritative. Until review confirmation, the user MAY change an issue's
action or correction with another valid PATCH.

Valid actions:

```text
accept
edit
remove
unrecognized
```

`accept`, `remove`, and `unrecognized` MUST contain only `action`.

`edit` MAY contain only these editable fields and MUST contain at least one
actual correction:

```text
product_name
quantity
size_value
size_unit
line_total
```

`quantity`, when present, MUST be greater than 0. `size_value` and
`size_unit` are a pair: both MUST be present together, or both omitted.
Do not invent additional editable fields. The API does not recalculate
`line_total` from other submitted fields.

Request example:

```json
{
  "action": "edit",
  "product_name": "Coca-Cola Original",
  "quantity": 1,
  "size_value": 2.5,
  "size_unit": "L",
  "line_total": 44.0
}
```

After the analysis leaves `needs_review`, review mutations are invalid.

---

## Confirm review

```text
POST /analyses/{analysis_id}/review/confirm
```

Valid while analysis `status` is `needs_review`.

Permitted only when every review issue has a persisted resolution. If any
issue is still unresolved, the API MUST return `409 REVIEW_INCOMPLETE`
with machine-readable `details.unresolved_count` and
`details.unresolved_issue_ids`. Those identifiers are review issue IDs
(`review_…`); they MUST NOT include raw OCR, filenames, or purchase
payloads.

On success, the backend resumes the canonical analysis lifecycle and
returns the updated Analysis. Typical next `status` is `generating_report`.
Clients MUST use that authoritative resource and MUST NOT invent
`generating_report` locally.

After successful confirmation, further review PATCH or confirm requests
are invalid. Unknown confirmation outcomes MUST be recovered by a fresh
authoritative `GET /analyses/{analysis_id}` (and `GET …/review` only if
status is still `needs_review`), not by blindly retrying the mutation.

---

## Get report

```text
GET /analyses/{analysis_id}/report
```

Response shape:

```json
{
  "analysis_id": "ana_xxx",
  "access": "free",
  "period": {
    "from": "2026-08-22",
    "to": "2026-09-07"
  },
  "summary": {},
  "categories": [],
  "personality": {},
  "basic_savings": {},
  "share_payload": {},
  "full": null,
  "upgrade": {
    "available": true,
    "price_mxn": 49.00,
    "interval": "month",
    "plan": "one"
  }
}
```

For an owner with an active verified One period:

```text
access = "full"
full != null
```

The account-level subscription routes are `POST /subscriptions/checkout`,
`GET /subscriptions` and `POST /subscriptions/cancel`. Checkout requires a
verified permanent account and an idempotency key. Price and plan are selected
by the server. Only provider-verified payment grants access. Cancellation ends
renewal while preserving already-paid access. The API never returns provider
payment references or card data.

`GET /history` provides bounded, cursor-paginated owned purchases.
`PATCH /purchases/{item_id}` corrects a sanitized owned purchase and invalidates
its month and following month's report. `GET /products/{product_id}/prices`
returns only the caller's comparable historical receipt observations.
`GET /reports/monthly?month=YYYY-MM` reports observed receipts in the owner
timezone, exposes dated/undated and category coverage, includes an unclassified
bucket, and compares only against the previous calendar month when coverage is
sufficient. These operations and their exact response shapes are normative in
OpenAPI.

Generating a new monthly report requires active One access. A monthly report
already generated remains readable after cancellation; it is not recomputed
without active access. A report earned through a verified paid period likewise
remains readable after that period ends.

`POST /me/deletion` requires explicit `confirm: true` and creates an idempotent
asynchronous deletion request; `GET /me/deletion` returns its owner-only status.
It purges account product data, reports, aliases, caches and in-flight raw
objects. Only minimum legally required financial/audit rows may remain, with
identity removed and product access revoked. A request status is not evidence
that storage deletion has completed.

---

# 25. Error Contract

All API errors use:

```json
{
  "error": {
    "code": "RECEIPT_LIMIT_REACHED",
    "message": "Este análisis ya alcanzó el máximo de tickets.",
    "details": {}
  }
}
```

Common codes:

```text
UNAUTHORIZED
FORBIDDEN
ANALYSIS_NOT_FOUND
INVALID_FILE_TYPE
FILE_TOO_LARGE
RECEIPT_LIMIT_REACHED
UPLOAD_NOT_FOUND
PROCESSING_ALREADY_STARTED
PROCESSING_FAILED
REVIEW_REQUIRED
REVIEW_INCOMPLETE
REPORT_NOT_READY
PAYMENT_REQUIRED
PAYMENT_FAILED
RATE_LIMITED
VALIDATION_ERROR
INTERNAL_ERROR
```

Do not expose stack traces to clients.

---

# 26. Report Contract

Free report minimum structure:

```json
{
  "summary": {
    "total_spent_mxn": 2847.23,
    "receipt_count": 3,
    "item_count": 63,
    "store_count": 2
  },
  "categories": [],
  "personality": {},
  "basic_savings": {},
  "share_payload": {}
}
```

Full report extends with:

```json
{
  "full": {
    "recurring_products": [],
    "store_comparison": [],
    "price_history": [],
    "unit_price_insights": [],
    "patterns": [],
    "detailed_savings": {},
    "insights": []
  }
}
```

---

# 27. Share Payload

API should provide sanitized factual data.

Example:

```json
{
  "template": "personality",
  "brand": "Lleno",
  "emoji": "🥩",
  "title": "El Norteño Constitucional",
  "primary_text": "La carne no es una categoría. Es infraestructura.",
  "stat": {
    "label": "de mi despensa fue proteína",
    "display_value": "31%"
  },
  "footer": "lleno.io"
}
```

The APP owns visual rendering/export of the card.

API owns safe/factual content.

---

# 28. Deterministic Analytics

Backend must contain deterministic functions for:

- total spent
- receipt count
- item count
- store count
- date range
- category totals
- category percentages
- average ticket
- normalized unit price
- recurring-product frequency
- observed minimum price
- observed maximum price
- historical modeled savings
- store-specific observed prices

Calculations should have automated tests.

Monetary values should use appropriate decimal arithmetic.

Do not use binary floating point as the authoritative money representation.

---

# 29. Observability

Track high-level processing metadata:

- job state
- latency
- OCR engine used
- fallback used
- confidence scores
- item counts
- error classes

Do not log sensitive receipt content.

Use correlation IDs.

---

# 30. Analytics Events

Canonical event names:

```text
app_opened
anonymous_session_created
analysis_created
upload_started
receipt_uploaded
processing_started
processing_completed
processing_failed
review_opened
review_completed
wrapped_viewed
personality_viewed
share_started
share_completed
upgrade_viewed
checkout_started
payment_completed
full_report_viewed
additional_receipts_uploaded
account_conversion_started
account_conversion_completed
subscription_checkout_started
subscription_started
subscription_renewed
subscription_payment_failed
subscription_canceled
receipt_quota_exhausted
purchase_corrected
monthly_report_viewed
```

These events are server-observed. The contract does not define a public event
ingest endpoint. Event payloads contain only approved aggregate identifiers and
never receipt content or payment/provider references.

---

# 31. Contract Repository

Create a lightweight repository:

`lleno-contract`

Suggested structure:

```text
lleno-contract/
├── README.md
├── VERSION
├── docs/
│   ├── MVP.md
│   ├── STATE_MACHINE.md
│   ├── PRIVACY_RULES.md
│   └── PRODUCT_RULES.md
├── openapi/
│   └── openapi.yaml
├── schemas/
│   ├── analysis.schema.json
│   ├── report.schema.json
│   ├── personality.schema.json
│   └── share-payload.schema.json
└── fixtures/
    ├── free-report.json
    ├── full-report.json
    ├── processing-analysis.json
    ├── needs-review.json
    ├── failed-analysis.json
    ├── me-free.json
    ├── subscription-active.json
    ├── purchase-history.json
    ├── monthly-report.json
    └── personal-price-history.json
```

Both codebases pin a known contract version.

Example:

`LLENO_CONTRACT_VERSION=0.2.1`

---

# 32. Parallel Development Rule

The APP developer may use fixtures/mock API responses while API development is incomplete.

The API developer may use contract tests without a frontend.

Integration is complete only when:

```text
app generated client
       ↓
canonical OpenAPI
       ↑
API implementation
```

No manually duplicated TypeScript API interfaces should become a second source of truth.

---

# 33. MVP Acceptance Scenario

A brand-new mobile user must be able to:

1. open app.lleno.io
2. begin without entering personal information
3. upload three receipt images
4. see processing progress
5. correct ambiguous items if needed
6. obtain a complete free Wrapped
7. see a funny data-grounded personality
8. export/share a safe social card
9. see that deeper analysis exists
10. convert to a verified recoverable account before checkout
11. start Lleno One at MXN $49.00 per monthly provider billing cycle
12. receive paid access only after a verified provider payment
13. use the remaining 100-document monthly allowance across bounded analysis batches
14. search owned purchases, view personal observed prices and read an on-demand monthly report
15. cancel renewal while keeping access through the paid-through timestamp
16. request account deletion and check its completion status

Original receipt images must not survive beyond the configured five-hour maximum.

---

# 34. Product Principle

When choosing between complexity and user delight:

Prioritize the smallest implementation that delivers:

```text
Curiosity
   ↓
Understanding
   ↓
Surprise
   ↓
Share
   ↓
Deeper value
   ↓
Payment
```

Do not build features outside the frozen MVP unless explicitly approved.
