# MVP

Frozen scope for the first production version of Lleno.

Market: Mexico. Language: Spanish. Currency: MXN.

## Product definition

Lleno converts everyday consumer receipts into personal consumer intelligence.

The first product is **Despensa Wrapped**.

A user uploads receipts from purchases made in Mexico. Lleno:

1. reads the receipts
2. removes unnecessary sensitive information
3. extracts purchases
4. normalizes products
5. calculates deterministic spending statistics
6. discovers patterns
7. generates playful consumer-personality content
8. produces a visual, interactive Wrapped
9. upgrades the owner's account to One through a recurring monthly subscription

Lleno is **not** an expense tracker.

Lleno is: consumer intelligence for ordinary people.

## Core funnel

```text
lleno.io
    ↓
app.lleno.io
    ↓
anonymous session
    ↓
create analysis
    ↓
upload receipt documents within monthly allowance
    ↓
process
    ↓
light review if necessary
    ↓
FREE WRAPPED
    ↓
share
    ↓
use monthly receipt allowance and owned history
```

Primary business metric: `free_wrapped_to_paid_conversion`  
Primary organic-growth metric: `wrapped_share_rate`

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

## User model

The first analysis MUST NOT require:

- email
- password
- name
- account creation form

The platform establishes an anonymous authenticated identity automatically.

Recommended mechanism: Supabase Anonymous Auth.

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

Clients MUST NOT submit arbitrary `user_id` ownership fields. The API derives identity from the bearer token and verifies household/analysis ownership server-side.

## Limits

| Constraint | Free | One |
| --- | --- | --- |
| Analyses a user may create | Multiple free analyses allowed | Account-level One access |
| Receipt documents per analysis/job | up to 15 | up to 15 |
| Formats | JPG, JPEG, PNG, PDF | same |
| Monthly successful unique receipts | 5 | 100 initial beta ceiling |
| Price | none | MXN $49.00 per monthly provider billing cycle |

Do **not** implement a lifetime "one free analysis per person" restriction in MVP.

Reasonable abuse protection and rate limits are allowed.

Receipts may come from any consumer purchase: supermarkets, convenience stores, pharmacies, warehouse clubs, local stores, pet stores, household goods, food-related businesses, and other everyday consumer merchants.

Do not restrict the parser architecture to known retailers.

## Technical boundaries

### lleno.io

Public marketing/editorial website. Separate existing project.

Responsibilities: SEO, articles, acquisition, product explanation, CTA to `app.lleno.io`.

Not part of this implementation contract beyond URLs.

### app.lleno.io

Frontend product.

Recommended: React, TypeScript, Vite, Tailwind, TanStack Query, React Router, Zod, React Hook Form, Motion where useful.

No direct database access for product data.

### api.lleno.io

Application API/orchestrator.

Recommended: TypeScript, Cloudflare Workers, Hono, OpenAPI, Supabase PostgreSQL/Auth, Cloudflare R2, Cloudflare Queues.

Base URL: `https://api.lleno.io/v1`

### Receipt processor

Background processing service.

Recommended: Python, Docker, OpenCV, OCR engine abstraction, Cloudflare Containers.

Primary OCR path should be inexpensive/open-source (PaddleOCR, Tesseract). Engine choice must be replaceable. Fallback on confidence or parse failure.

## Core database entities

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

## Explicitly out of MVP

Do **not** implement:

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

## Analytics events

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
```

## Observability

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

## MVP acceptance scenario

A brand-new mobile user must be able to:

1. open `app.lleno.io`
2. begin without entering personal information
3. upload three receipt images
4. see processing progress
5. correct ambiguous items if needed
6. obtain a complete free Wrapped
7. see a funny data-grounded personality
8. export/share a safe social card
9. see that deeper analysis exists
10. convert to a verified recoverable account before checkout
11. start Lleno One at MXN $49.00 per monthly provider cycle
12. receive paid access only after a verified provider payment
13. use up to the account's remaining 100-document monthly allowance across bounded analysis batches
14. search owned purchases, view personal observed prices, and read an on-demand monthly report
15. cancel renewal and retain access through the paid-through timestamp
16. request account deletion and inspect its completion status

Original receipt images must not survive beyond the configured five-hour maximum.

## Product principle

When choosing between complexity and user delight, prioritize the smallest implementation that delivers:

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
