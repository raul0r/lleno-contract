# lleno-contract

Canonical product and API contract for Lleno MVP.

**Status:** FROZEN FOR MVP IMPLEMENTATION  
**Version:** `0.1.3` (see [`VERSION`](./VERSION))
**Market:** Mexico  
**Language:** Spanish  
**Currency:** MXN

| Surface | Domain |
| --- | --- |
| Product | lleno.io |
| Web app | app.lleno.io |
| API | api.lleno.io |

Pin this version in both codebases:

```text
LLENO_CONTRACT_VERSION=0.1.3
```

## Authority

This repository is more authoritative than `lleno-app` or `lleno-api`.

Both implementations MUST implement against this contract.

Neither implementation may invent incompatible endpoints, fields, states, business rules, limits, or workflows.

The frozen narrative source is [`lleno-contract.md`](./lleno-contract.md). The files in `docs/`, `openapi/`, `schemas/`, and `fixtures/` are the implementation-facing split of that document. If a split file and `lleno-contract.md` disagree, treat `lleno-contract.md` as canonical, then fix the split files before changing either codebase.

## Change process

If implementation reveals a necessary contract change:

1. Update this contract first (`lleno-contract.md` and the matching split files).
2. Update OpenAPI, JSON schemas, and fixtures.
3. Increment [`VERSION`](./VERSION).
4. Only then modify app/API implementations.

## Layout

```text
lleno-contract/
├── README.md
├── VERSION
├── lleno-contract.md          # frozen narrative contract
├── docs/
│   ├── MVP.md                 # scope, funnel, acceptance, out of MVP
│   ├── PRODUCT_RULES.md       # free/paid product behavior
│   ├── STATE_MACHINE.md       # analysis/receipt states and pipeline
│   └── PRIVACY_RULES.md       # retention, PII, logging, share safety
├── openapi/
│   └── openapi.yaml           # HTTP API
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
    └── failed-analysis.json
```

## How agents should use this repo

### Frontend (`lleno-app`)

- Generate the API client from [`openapi/openapi.yaml`](./openapi/openapi.yaml). Do not hand-write a second TypeScript API interface layer that becomes a competing source of truth.
- Validate report, personality, and share payloads with the JSON schemas.
- Use [`fixtures/`](./fixtures/) as mock API responses until the API is ready.
- User-facing copy is Mexican Spanish. Personality and share text in fixtures is canonical tone, not final copy.
- First analysis must start from anonymous auth. No email, password, name, or account form.

### Backend (`lleno-api` and receipt processor)

- Implement the OpenAPI document exactly: paths, methods, status envelopes, and field names.
- Contract-test responses against the JSON schemas and fixtures.
- Arithmetic in reports is deterministic. AI must not be the source of truth for totals, percentages, unit prices, or savings.
- Schema-validate all structured AI outputs (`personality`, insights, share copy).
- Enforce privacy rules: original files are private, short-lived, and deleted; do not persist or log raw OCR, payment data, or loyalty identifiers.

### Parallel development

## Fixtures

Use these as mock API responses. They validate against the JSON schemas.

| File | Mock of |
| --- | --- |
| [`fixtures/processing-analysis.json`](./fixtures/processing-analysis.json) | `GET /analyses/{analysis_id}` while `processing` |
| [`fixtures/needs-review.json`](./fixtures/needs-review.json) | `GET /analyses/{analysis_id}` while `needs_review` |
| [`fixtures/failed-analysis.json`](./fixtures/failed-analysis.json) | `GET /analyses/{analysis_id}` while `failed` |
| [`fixtures/free-report.json`](./fixtures/free-report.json) | `GET /analyses/{analysis_id}/report` with `access=free` |
| [`fixtures/full-report.json`](./fixtures/full-report.json) | `GET /analyses/{analysis_id}/report` with `access=full` |

Review-issue payloads live as examples on `GET /analyses/{analysis_id}/review` in the OpenAPI document.

## Parallel development

The app may use fixtures/mock responses while the API is incomplete.

The API may use contract tests without a frontend.

Integration is complete only when:

```text
app generated client
       ↓
canonical OpenAPI
       ↑
API implementation
```

## Product in one line

Lleno converts everyday consumer receipts into personal consumer intelligence.

The MVP product is **Despensa Wrapped**. Lleno is not an expense tracker.

Primary business metric: `free_wrapped_to_paid_conversion`  
Primary organic-growth metric: `wrapped_share_rate`
