# REST API Checklist

The contract for `@review-rest-api`. **Activation rule**: apply only when the diff creates or modifies HTTP API surface — routes, views, handlers, serializers, request/response schemas, URL config. If the diff touches none of these, output `## REST API review\nNo API surface in diff — skipped.` and stop.

For every finding: cite `file:line`, explain why, give a concrete fix.

## 1. Resource modeling (URLs)

- [ ] Nouns, not verbs — `/companies`, not `/getCompany`. The URL names the thing; the method names the action.
- [ ] Plural nouns for collections — `/candidates`, `/notes`. Consistent plurality everywhere.
- [ ] Resource IDs in the path — `/companies/{companyId}`, never `/companies/get?id=...`.
- [ ] Hierarchy for real relationships — `/projects/{projectId}/people/{personId}`. Limit nesting to ~2 levels; deeper belongs in query params.
- [ ] Lowercase, hyphenated paths — `/pipeline-statuses`, not `/pipelineStatuses` or `/pipeline_statuses`.
- [ ] No file extensions — `/candidates/123`, not `/candidates/123.json`. Format negotiation goes in headers.
- [ ] Stable, opaque identifiers — UUIDs or IDs, never sequential hints of internals if avoidable; never change a URL once shipped.
- [ ] Version in the path — `/v2/companies`, versioned as a whole, not per-endpoint.

## 2. HTTP method semantics

- [ ] GET = read only. Safe, idempotent, zero side effects. Never mutate on GET.
- [ ] POST = create on a collection (`POST /companies`), or a genuine non-CRUD action (`POST /companies/{id}/merge`) — actions are the exception, not the default.
- [ ] PUT = full replace of a known resource (idempotent).
- [ ] PATCH = partial update. Use it — don't funnel every edit through POST.
- [ ] DELETE = delete. `DELETE /companies/{id}`, not `POST /companies/delete`.
- [ ] Idempotency respected — GET/PUT/DELETE repeatable without damage; support idempotency keys on POST for payments/critical writes.
- [ ] OPTIONS/HEAD/CORS handled correctly — preflight support where browsers need it.

## 3. Querying collections

- [ ] Filtering via query params — `GET /people?companyId=abc&status=active`.
- [ ] Sorting — `?sort=-createdAt,name`.
- [ ] Pagination always — cursor-based (`?cursor=xyz&limit=50`) for large/changing datasets; offset (`?page=`) acceptable for small stable ones. Never unbounded lists.
- [ ] Field selection — `?fields=id,name,company` to trim payloads.
- [ ] Expansion for relations — `?expand=company` when the client genuinely needs the join, instead of forcing N extra calls.
- [ ] Search is a GET on the collection (`GET /people?q=...`) — not `POST /query`. Reads belong on GET so they're cacheable, bookmarkable, and shareable.

## 4. Request/response bodies

- [ ] JSON everywhere, `Content-Type: application/json`, camelCase or snake_case — pick one and never mix.
- [ ] Consistent envelope — either bare resources or one standard wrapper; same shape for collections (`{items: [], nextCursor: ...}`) everywhere.
- [ ] Return the created/updated resource — POST/PATCH/PUT respond with the full resulting object, not just "ok".
- [ ] Don't leak internals — no raw DB rows, stack traces, or internal field names in responses.
- [ ] Nulls and empty lists are explicit — `[]` for empty collections, never omit the key.
- [ ] Consistent date format — ISO 8601 UTC everywhere.

## 5. Status codes — use them honestly

- [ ] 2xx: 200 OK, 201 created, 202 accepted (async), 204 no content (deletes).
- [ ] 4xx: 400 validation, 401 unauthenticated, 403 unauthorized, 404 missing, 409 conflict (e.g. duplicate), 422 semantic errors, 429 rate limited.
- [ ] 5xx only for real server faults — never return 200 with `{"error": ...}` in the body.
- [ ] Consistent error schema — one shape: `{code, message, details?}`, machine-readable codes, human-readable messages, field-level validation errors enumerated.
