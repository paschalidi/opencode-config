# Testing Checklist

The contract for `@review-python-tests`. Full standard (wins on any conflict): `~/.config/opencode/instructions/python-testing-standards.md`.

For every finding: cite `file:line`, explain why, give a concrete fix.

## Activation rule — tests in the diff only

The target codebase has no test suite. **Missing tests are never a finding.**

- The diff adds or changes **no test files** → output `## Tests review\nNo tests in diff — skipped.` and stop. Never suggest writing tests.
- The diff **does** add or change tests → review those tests against this checklist, and you may suggest extra cases for the behaviour they already cover.
- **Every test finding is non-blocking** — never Critical, never `blocking`. Use Important or Nice-to-have.

## Coverage — only for tests the PR adds or changes

- [ ] The tests exercise the changed branches and edge paths, not just the happy path → suggest the missing cases (non-blocking).

## Survives refactoring — the most important section

- [ ] Tests go through a **public interface** (endpoint, service function, handler's public method). Calling private methods or internal helpers → flag.
- [ ] Tests assert **results or stored state**, not internal calls. `assert_called_*` on your own code → flag (change-detector test). Allowed only when the call is the behaviour: email, message, webhook, external audit event.
- [ ] A behaviour-preserving refactor (rename, extract, merge functions) would leave the test green. If not → flag and say which refactor breaks it.

## Test doubles

- [ ] Order of preference: real → fake → stub → mock.
- [ ] Own code is never mocked. Own database is the real test DB or an in-memory fake, not mocked.
- [ ] Third-party APIs, email, queues: stubs for queries, mocks only for commands whose side effect is the behaviour; ideally through a thin adapter.
- [ ] Doubles live in a shared fixture; more than ~3 `@patch` decorators on one test → flag: extend the fixture or move the test.
- [ ] Clock, randomness and ids are frozen or injected.

## Layer (Django apps)

- [ ] Model tests: DB constraints and Pydantic validation only.
- [ ] Service tests: permissions (denied and allowed), not-found, resulting state, external side effects. Asserting internal calls → flag.
- [ ] Handler tests: computation scenarios through public `compute()`.
- [ ] HTTP tests: 401, permission → status mapping, error shape, one happy-path contract test. Business rules re-tested here → flag.
- [ ] The same assertion is not repeated across layers.

## Readability

- [ ] One behaviour per test; the name states scenario and outcome (`test_permission_denied_when_user_has_wrong_role`), no "and".
- [ ] Given / When / Then visible (blank lines or comments); no interleaved act/assert.
- [ ] Straight-line code: no loops, ifs or computed expectations.
- [ ] Values the assertion depends on appear in the test, not hidden in shared constants or setup.
- [ ] Factories for DB records, builders for external API payloads — never `Model.objects.create(...)` or hand-crafted API JSON.
- [ ] `@pytest.mark.parametrize` for variations of one behaviour, with readable `ids`; no copy-paste tests; no mixing different behaviours in one table.
- [ ] pytest style with plain `assert` on exact values.
- [ ] File names follow `test_{service_function}.py`.

## Isolation & edge cases

- [ ] Tests run alone and in any order; no shared mutable state; no real network.
- [ ] Edge cases: empty results, invalid inputs (`""`, unknown, `None`), boundary values.
- [ ] New endpoints/services: denied and allowed permission paths.
