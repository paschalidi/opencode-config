# Testing Checklist

The contract for `@review-python-tests`. Full standard (wins on any conflict): `~/.config/opencode/instructions/python-testing-standards.md`.

For every finding: cite `file:line`, explain why, give a concrete fix.

## Coverage

- [ ] New or changed logic ships with tests. Missing tests for new behavior → Important finding (Critical if the logic touches money, auth, or personal data).
- [ ] Changed branches and edge paths are covered, not just the happy path.

## Right layer — test at the lowest layer where the behavior lives

- [ ] Model tests verify only DB constraints and Pydantic validation. Nothing else.
- [ ] Service tests prove **orchestration, not outcomes**: `PermissionDenied` for the right (and wrong) permissions, `NotFoundError` for unknown ids, audit logging called with the right arguments, business-logic function **called** (not what it returned). A service test asserting computed values → flag; it belongs at the handler layer.
- [ ] Handler tests exercise the **public** `compute()` only — happy path plus all edge cases. Testing private methods → flag.
- [ ] HTTP tests cover only the four concerns: 401 without auth, permission → status-code mapping, error response shape, one happy-path contract test per endpoint. Computation, filtering, and permission rules re-tested at HTTP layer → flag.
- [ ] The same logical assertion is not duplicated across layers.

## Data & mocks

- [ ] DB records via factories (`ItemFactory.create(...)`) — never raw `Model.objects.create(...)` in tests.
- [ ] External API JSON via builders — never hand-crafted response dicts from scratch.
- [ ] Domain result objects not hand-built to satisfy a mock — that test belongs at the handler layer.
- [ ] Mocks only at real network boundaries (shared `mock_all_external_apis`-style fixture as baseline). Internal methods never mocked.
- [ ] More than ~5 `@patch` decorators on one test → flag: extend the shared fixture instead.

## Shape

- [ ] pytest style with plain `assert`.
- [ ] Variations via `@pytest.mark.parametrize` — no copy-paste tests that change one variable.
- [ ] Test names describe the scenario and expected outcome (`test_permission_denied_when_user_has_wrong_category`), not the implementation (`test_status_filter`).
- [ ] File names follow `test_{service_function}.py`.
- [ ] Permissions tests for every new endpoint/service function — both denied and allowed paths.
- [ ] Edge cases: empty results, invalid inputs (parametrized `""`, invalid, `None`), boundary values.
- [ ] Tests assert through the public interface, not past it.
