# Product rules

Normative product behavior for Lleno MVP. User-facing language is Mexican Spanish. Currency is MXN.

## Free tier

Free users may create multiple free analyses.

Do **not** implement a lifetime "one free analysis per person" restriction in MVP.

Reasonable abuse protection and rate limits are allowed.

Each FREE analysis supports `1–3` receipts.

Supported formats: JPG, JPEG, PNG, PDF.

Receipts may come from any consumer purchase. Do not restrict the parser to known retailers.

## Free report

The free Wrapped MUST provide real value. Do not intentionally make the free experience useless.

Free report includes:

### Summary

- total amount spent
- number of receipts
- number of products/items analyzed
- detected date range
- stores detected

### Categories

- category totals
- category percentages
- top category

### Personality

A creative, funny, data-grounded "Despensa Personality".

This is a CORE acquisition feature.

### Basic savings insight

At least one simple savings or price observation when supported by the user's own data.

Examples:

- same normalized product purchased at different prices
- same category showing meaningful price variation
- different versions/brands of similar products purchased at different normalized unit prices

Never fabricate savings. If the user's data does not support an observation, omit it.

### Share card

A social card generated from sanitized report data.

Primary formats should work for:

- WhatsApp Status
- Instagram Stories
- TikTok/Reels screenshots
- Facebook

The APP owns visual rendering/export of the card.

The API owns safe/factual content (`share_payload`).

Default sharing MUST NOT expose:

- exact total spending
- payment details
- personal identifiers
- loyalty identifiers
- addresses
- receipt numbers
- card information

## Despensa personality

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

All evidence values must come from deterministic analytics.

Shape: see [`schemas/personality.schema.json`](../schemas/personality.schema.json) and [`schemas/share-payload.schema.json`](../schemas/share-payload.schema.json).

## Complete analysis (paid)

The initial paid product is a **one-time purchase**. No subscription is part of MVP.

Price: approximately USD $6, charged as a configurable fixed MXN amount.

Configuration: `FULL_REPORT_PRICE_MXN`

Do **not** calculate the product price dynamically from FX rates.

The operator must be able to change the MXN price without redeploying application logic.

Paid entitlement applies to **one analysis**.

Maximum: `15 receipts per full analysis`.

A user who pays after analyzing 3 receipts may:

- keep the existing 3
- add up to 12 additional receipts
- reprocess the same analysis
- produce an improved Full Wrapped

Payment provider implementation belongs to the API.

Payment callbacks/webhooks MUST be verified server-side.

Payment success creates entitlement.

## Full report

Everything from Free plus:

### Product recurrence

Identify products purchased repeatedly.

### Product normalization

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

Never silently merge low-confidence products.

Maintain: canonical product, aliases, normalization confidence, normalization source, review status.

Human corrections should become useful alias evidence later.

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

### Unit normalization

Where possible normalize:

- ml → L
- g → kg
- unit count
- packs
- price per unit
- price per liter
- price per kilogram

### Price history

Compare prices observed in the user's own receipts.

No external retailer price lookup in MVP.

### Store comparison

Compare observed prices only when data supports a meaningful comparison.

### Detailed savings opportunities

Example:

> If recurring products had been purchased at the lowest normalized unit price you personally observed during this period, the modeled difference would have been approximately $X.

Language must clearly indicate this is historical/modelled, not guaranteed savings.

### Patterns

Examples:

- recurring products
- category concentration
- store concentration
- average ticket
- higher/lower spending contexts supported by dates
- repeated price differences
- product alternatives already purchased by the user

### Deeper insight cards

AI may describe patterns, but numbers come from deterministic computation.

## Review experience

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

User can:

- accept
- edit
- mark unrecognized
- remove line

After confirmation, analysis continues.

## AI boundaries

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

## Deterministic analytics

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

JSON responses may serialize money as decimal numbers with two fractional digits (see fixtures). Internal storage and computation must not treat IEEE-754 floats as the ledger.

## Report access

| Field | Free | Full |
| --- | --- | --- |
| `access` | `"free"` | `"full"` |
| `full` | `null` | object |
| `upgrade.available` | `true` | `false` |

Entitlement response after payment:

```json
{
  "access": "full",
  "max_receipts": 15,
  "payment_status": "paid"
}
```

## Share payload

API provides sanitized factual data. APP renders the card.

Default template is personality-led. Footer is `lleno.io`. Brand is `Lleno`.

See [`schemas/share-payload.schema.json`](../schemas/share-payload.schema.json).
