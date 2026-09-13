# REST API Checklist

The contract for `@review-rest-api`. **Activation rule**: apply only when the diff creates or modifies HTTP API surface — routes, views, handlers, serializers, request/response schemas, URL config. If the diff touches none of these, output `## REST API review\nNo API surface in diff — skipped.` and stop.

For every finding: cite `file:line` (and the rule id), explain why, give a concrete fix. Rules are testable: if you cannot point at the violating line, do not flag it; if the PR description justifies a deviation, weigh the justification instead of auto-flagging.

## How to apply

1. **House rules R1–R25 first** — the strict, non-negotiable layer. Where a house rule and a generic item overlap, the house rule wins.
2. **Then the generic best practices** — they extend the house rules (casing, envelopes, idempotency keys, CORS, …).

Severity: R7, R18, R21, R23 are **Critical** (side-effecting reads, dishonest status codes, broken object-level authz, secrets in URLs). Everything else defaults to **Important** unless blast radius justifies higher.

---

## PART 1 — HOUSE RULES

### 1.1 Resource modeling
- [ ] **R1.** URLs contain NOUNS, never verbs. `/companies/{id}` yes; `/companies/delete`, `/candidates/push_to_crm` no.
- [ ] **R2.** New endpoints MUST place resource IDs in the path: `/projects/{projectId}/people/{personId}`. IDs in query strings or POST bodies for single-resource access are a violation.
- [ ] **R3.** Plural nouns for collections, lowercase, hyphenated (`/pipeline-statuses`, not `pipelineStatuses`).
- [ ] **R4.** Nest at most 2 levels; deeper relationships become query params.
- [ ] **R5.** Non-CRUD actions are allowed ONLY as `POST /{resource}/{id}/{action}` (e.g. `POST /companies/{id}/merge`) and must be justified — actions are the exception, not the default pattern.
- [ ] **R6.** Version APIs in the path (`/v2/...`). Within a version, changes are additive-only; removing/renaming response fields requires a new version.

### 1.2 HTTP method semantics
- [ ] **R7.** GET is read-only. ZERO side effects — no writes, no feed-building, no state mutation inside a GET handler. (Existing offender: `GET /activities/fill_feed` — never add another.)
- [ ] **R8.** Reads use GET, always. `POST /x/query`, `POST /x/get` are violations — reads must be cacheable/bookmarkable.
- [ ] **R9.** Deletes use DELETE (or POST action per R5, never `POST /x/delete` on the internal pattern for NEW endpoints). Updates use PATCH (partial) or PUT (full replace). Create uses POST on the collection.
- [ ] **R10.** One route, one method. No GET/POST twins of the same path.
- [ ] **R11.** `save` endpoints that hide create-vs-update behind one verb are flagged; prefer explicit POST (create) + PATCH (update).

### 1.3 Querying collections
- [ ] **R12.** Filtering/sorting/pagination via query params on GET collections: `GET /people?companyId=x&sort=-createdAt&cursor=y&limit=50`.
- [ ] **R13.** Every list endpoint MUST paginate — no unbounded result sets.
- [ ] **R14.** Cursor-based pagination (opaque signed token), not offset, for anything large or changing. Offset capped at 10k is a smell.
- [ ] **R15.** Cross-resource reads the UI always needs together get an `?expand=` param or a purpose-built composite endpoint — NOT forcing the frontend into N+1 parallel calls. No more `companies_callback`-style patch-up endpoints that exist only to fix a join the query skipped.

### 1.4 Responses & errors
- [ ] **R16.** JSON only, one casing convention, one envelope shape for lists (`{items, nextCursor}`) everywhere.
- [ ] **R17.** POST/PATCH/PUT return the resulting resource, not a bare ack.
- [ ] **R18.** Status codes are honest: 201 created, 202 accepted (async), 204 deleted, 400 validation, 401/403 auth, 404 missing, 409 conflict, 422 semantic, 429 rate-limited. NEVER 200 with an error body.
- [ ] **R19.** One machine-readable error schema: `{code, message, details?}` with field-level validation errors enumerated.
- [ ] **R20.** Dates ISO 8601 UTC everywhere. Empty collections are `[]`, never omitted keys. No internal field names or raw rows leaked.

### 1.5 Security & performance
- [ ] **R21.** Object-level authz on every endpoint: verify the requested entity belongs to the caller's team. Endpoint-level auth alone fails.
- [ ] **R22.** Rate limiting on public/expensive endpoints with 429 + Retry-After.
- [ ] **R23.** No sensitive data in URLs (tokens, PII) — URLs land in logs.
- [ ] **R24.** Slow work (>a few seconds) is async: 202 + job resource, not a held-open request near the 29s gateway timeout.
- [ ] **R25.** GETs set deliberate Cache-Control; consider ETag/304 where stale-reads are acceptable.

---

## PART 2 — GENERIC BEST PRACTICES

### Resource modeling (URLs)
- [ ] Nouns, not verbs — the URL names the thing; the method names the action.
- [ ] Plural nouns for collections — consistent plurality everywhere.
- [ ] Resource IDs in the path, never `/companies/get?id=...`.
- [ ] Hierarchy for real relationships; limit nesting to ~2 levels.
- [ ] Lowercase, hyphenated paths.
- [ ] No file extensions — `/candidates/123`, not `/candidates/123.json`. Format negotiation goes in headers.
- [ ] Stable, opaque identifiers — UUIDs or IDs, never sequential hints of internals if avoidable; never change a URL once shipped.
- [ ] Version in the path, versioned as a whole, not per-endpoint.

### HTTP method semantics
- [ ] GET = read only. Safe, idempotent, zero side effects.
- [ ] POST = create on a collection, or a genuine non-CRUD action — actions are the exception, not the default.
- [ ] PUT = full replace of a known resource (idempotent).
- [ ] PATCH = partial update. Don't funnel every edit through POST.
- [ ] DELETE = delete.
- [ ] Idempotency respected — GET/PUT/DELETE repeatable without damage; support idempotency keys on POST for payments/critical writes.
- [ ] OPTIONS/HEAD/CORS handled correctly — preflight support where browsers need it.

### Querying collections
- [ ] Filtering via query params.
- [ ] Sorting — `?sort=-createdAt,name`.
- [ ] Pagination always; cursor-based for large/changing datasets, offset acceptable for small stable ones. Never unbounded lists.
- [ ] Field selection — `?fields=id,name,company` to trim payloads.
- [ ] Expansion for relations — `?expand=company` when the client genuinely needs the join.
- [ ] Search is a GET on the collection (`GET /people?q=...`) — not `POST /query`.

### Request/response bodies
- [ ] JSON everywhere, `Content-Type: application/json`, camelCase or snake_case — pick one and never mix.
- [ ] Consistent envelope — either bare resources or one standard wrapper; same shape for collections everywhere.
- [ ] Return the created/updated resource — not just "ok".
- [ ] Don't leak internals — no raw DB rows, stack traces, or internal field names in responses.
- [ ] Nulls and empty lists are explicit — `[]` for empty collections, never omit the key.
- [ ] Consistent date format — ISO 8601 UTC everywhere.

### Status codes — use them honestly
- [ ] 2xx: 200 OK, 201 created, 202 accepted (async), 204 no content (deletes).
- [ ] 4xx: 400 validation, 401 unauthenticated, 403 unauthorized, 404 missing, 409 conflict, 422 semantic errors, 429 rate limited.
- [ ] 5xx only for real server faults — never 200 with `{"error": ...}` in the body.
- [ ] Consistent error schema — one shape: `{code, message, details?}`, machine-readable codes, human-readable messages, field-level validation errors enumerated.

---

## QUICK PASS — API changes (final sweep)

- [ ] Noun path, IDs in path, no verb-in-URL for new endpoints
- [ ] Correct method (GET=read-only, PATCH/PUT/DELETE for mutations)
- [ ] Reads cacheable (no POST-for-read)
- [ ] Paginated + cursor-based; one envelope; honest status codes
- [ ] Object-level team authz verified
- [ ] No new GET twins / no new `save` conflations
