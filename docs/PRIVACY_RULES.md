# Privacy rules

Normative rules for original files, PII, logging, and share safety.

## Original receipt storage

Original receipt files live in private object storage.

Recommended: Cloudflare R2.

Original receipt files MUST:

- never be public
- use unguessable object keys
- be accessible only through short-lived signed access when required
- be deleted as soon as successful processing makes them unnecessary
- always have a hard expiry no later than **5 hours after R2 object creation**

Target:

```text
successful receipt
→ delete original shortly after successful extraction

failed/stalled/unconfirmed receipt
→ force delete at R2 object_created_at + 5h
```

The five-hour limit is a **maximum** retention period, not the normal desired retention period. Confirmation time cannot extend it. Application cleanup must delete by the R2 `object_created_at + 5 hours` deadline, including uploaded-but-unconfirmed objects and orphan signed PUTs. R2 lifecycle deletion is asynchronous and is only a coarse backstop; do not treat its policy as proof of a five-hour deadline. Cleanup runs with margin and alerts on failures or overdue objects.

Browser uploads directly to R2 via signed URLs issued by the API. The API validates object existence on `POST /analyses/{analysis_id}/uploads/complete`.

Deletion of the raw object does not delete extracted structured purchase data.

MVP acceptance: original receipt images must not survive beyond the configured five-hour maximum.

## Sensitive information

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

Store only data necessary for the product.

## Logging

Do not log:

- raw OCR
- full receipt image URLs
- payment data
- loyalty identifiers
- sensitive receipt content

Use correlation IDs.

Observability may track: job state, latency, OCR engine used, fallback used, confidence scores, item counts, error classes.

## AI inputs

AI receives structured statistics rather than raw receipts whenever possible.

Personality, insights, and share copy must be generated from sanitized, deterministic analytics — not from unsanitized OCR or original images.

## Share card

Default sharing MUST NOT expose:

- exact total spending
- payment details
- personal identifiers
- loyalty identifiers
- addresses
- receipt numbers
- card information

The API owns the sanitized `share_payload`. The app must not add forbidden fields onto the card.

## Auth and ownership

App establishes anonymous Supabase authentication.

API receives:

```http
Authorization: Bearer <supabase-jwt>
```

API validates the token and derives user identity from it.

Clients MUST NOT submit arbitrary `user_id` ownership fields.

Authorization must verify household/analysis ownership server-side.

The first analysis MUST NOT require email, password, name, or an account creation form.

## What may be stored durably

Allowed examples: sanitized line items, normalized products, aliases, categories, stores, deterministic aggregates, reports, personalities, entitlements, analytics event names and non-sensitive metadata.

Not allowed as durable data: original images/PDFs past the deletion policy, complete raw OCR, payment PAN/references, loyalty identifiers, addresses, names from tickets, card suffixes, authorization codes.

## Personality and copy safety

Personality content must not infer or mock income, poverty, health, body weight, addiction, religion, politics, ethnicity, sexual orientation, pregnancy, medical conditions, relationship status, or other sensitive traits.

See [PRODUCT_RULES.md](./PRODUCT_RULES.md).
