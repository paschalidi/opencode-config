# Data Modeling Checklist — DynamoDB / multi-table

The contract for `@review-data-modeling`. **This is the highest-priority axis of the review.** These rules are non-negotiable.

> Apply every rule when reviewing OR implementing code. Rules are testable:
> if you cannot point at the violating line, do not flag it; if the PR
> description justifies a deviation, weigh the justification instead of
> auto-flagging.

## Applicability

Applies when the diff touches DynamoDB access or data modeling — look for SDK helpers (`ddb_get`, `ddb_set`, `ddb_update`, `ddb_query`, `ddb_query_page`, `ddb_scan`, `multi_thread`, `sign_cursor`/`verify_cursor`), table definitions (`storage.yaml`, `AWS::DynamoDB::GlobalTable`), stream consumers, or fields copied between tables. None present → output `## Data modeling review\nNo DynamoDB/data-modeling surface in diff — skipped.` and stop.

## Severity defaults

- **Critical**: D1, D13, D15, D19 — data integrity and data loss. D12 violations that expose raw `LastEvaluatedKey` or skip ownership validation are also Critical (authz).
- **Important**: everything else, unless the blast radius justifies higher.

---

## 1. Data modeling

- [ ] **D1.** Snapshot point-in-time fields (names, prices, stage at event time) into event/child records at creation. Copies on audit-trail tables are immutable snapshots; copies on live records are maintained copies and MUST have a documented re-sync path (write fan-out or stream).
- [ ] **D2.** Before adding a cross-table read join, ask: should this be (a) a maintained denormalized copy, (b) a shared-PK item collection for a genuinely hot 1:N aggregate, or (c) an OpenSearch denormalized field? Pick deliberately; record the decision in a comment.
- [ ] **D3.** Item collections (shared partition key + distinct sort keys) are reserved for measured bottlenecks — do not introduce speculatively, and do not collocate independent entities.
- [ ] **D4.** Wide items: no unbounded embedded lists (`locations`, `templates`, legacy `activities` pattern). New 1:N child data goes in child tables. Watch the 400KB ceiling; split oversized attributes to a large-properties table like the existing `user-large-properties` pattern.
- [ ] **D5.** Every new denormalized field = a stream/fan-out sync path = drift risk. Minimize the count; each one needs a repair/backfill story.
- [ ] **D6.** Do NOT create new hand-rolled index tables (`candidate-index-*` pattern) — use a GSI or an OpenSearch field. Do NOT create a new cache table — use the existing `function-cache` + S3 overflow machinery.
- [ ] **D7.** New tables require justification vs. consolidating into an existing table with a composite key. Table count is a cost: IAM, streams, backups, GlobalTable replication, on-call load. Pure-cache tables must not be GlobalTables without a reason.

## 2. Access patterns

- [ ] **D8.** Cross-table fetches in a handler run in PARALLEL via the shared `multi_thread` helper — never sequential `get_*` chains. Sequential is acceptable ONLY when a later call depends on an earlier call's result (comment the dependency).
- [ ] **D9.** Fan-out reads pass `attributes=[...]` (projection) when only a few fields are consumed. Whole-item reads in loops are flagged.
- [ ] **D10.** Reference-data reads (teams, users, custom fields, CRM reference) go through cached getters (`@local_cache`, `@cache_team_data`) — never raw `ddb_get` on hot paths.
- [ ] **D11.** Batch reads use `ddb_get` batch (auto-chunked) or `ddb_scan` segments — never per-key loops for >10 items.
- [ ] **D12.** Client-facing list endpoints use `ddb_query_page` + signed opaque cursors (`sign_cursor`/`verify_cursor`) with ownership validation — never expose raw `LastEvaluatedKey`. Do not add new internal drain-all `ddb_query` paths behind APIs; unbounded internal drains need a cap and a comment.
- [ ] **D13.** Write-time consistency: when snapshotting related-record fields at write time (placements pattern), read the source with `consistent=True`.
- [ ] **D14.** Prefer `ddb_set` over `ddb_update`; conditional/atomic ops are the only justification for `ddb_update`. Never bypass the SDK helpers with raw boto3 in a lambda.

## 3. Boundaries & operations

- [ ] **D15.** Cross-domain writes (e.g. candidates flow writing to projects, comms, todos tables) must go through the owning domain's SDK write module — never `ddb_put` directly into another domain's table from a lambda. New direct cross-domain `ddb_put` calls are a violation.
- [ ] **D16.** New tables default PAY_PER_REQUEST (on-demand). Provisioned capacity requires measured justification.
- [ ] **D17.** Ephemeral data (tokens, OTPs, events, callbacks) gets a DDB TTL at table creation — no TTL is a flag.
- [ ] **D18.** Stream consumers: assume OpenSearch/index lag (1-2s). Endpoints must not read-through-the-index immediately after a write without handling the staleness (fallback to DDB read or UI-level pending state).
- [ ] **D19.** Every stream-triggered denormalization needs failure handling wired to the existing failed-lambda-invocation / failed-es-bulk-save retry path — fire-and-forget stream writes are a violation.

---

## Quick pass — data changes (final sweep)

- [ ] New field copied between tables? Sync path + repair story documented
- [ ] Parallel `multi_thread` for independent fetches; dependency commented
- [ ] `attributes=` projection on fan-out reads
- [ ] Cached getters for reference data
- [ ] No new index/cache table; no raw cross-domain `ddb_put`
- [ ] No unbounded embedded lists; TTL on ephemeral items
- [ ] Stream consumers have retry handling
