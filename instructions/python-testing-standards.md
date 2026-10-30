# Python Testing & Code Standards

## Code Style

### Type Hints

Always use type hints:

```python
# Good ✅
def list_items(
    call_context: CallContext,
    category: str,
    status: Literal["unresolved", "resolved", "pending", "all"] = "unresolved",
) -> QueryResults:
    ...

# Bad ❌
def list_items(call_context, category, status="unresolved"):
    ...
```

### Use Pydantic for Data Validation

```python
from pydantic import BaseModel, Field
from uuid import UUID

class ItemFilters(BaseModel):
    patient_id: UUID | None = Field(None, description="Filter by patient UUID")
```

### Service Layer Pattern

Keep business logic in services, not views:

```python
# services/list.py - Business logic here
def list_items(...) -> QueryResults:
    # Authorization, validation, business logic
    ...

# http_api/views.py - Thin HTTP layer
def list_items_view(request, filters):
    result = list_items(...)
    return map_to_response(result)
```

## Testing Guidelines

Distilled from Google's *Software Engineering at Google* (ch. 12–13), Kent Beck's Test Desiderata, Vladimir Khorikov's *Unit Testing Principles, Practices, and Patterns*, Martin Fowler (Practical Test Pyramid, GivenWhenThen), Ian Cooper ("TDD, Where Did It All Go Wrong") and the pytest docs.

The goal: a test fails when behaviour changes, and only then. A refactor that keeps behaviour must leave every test green.

### 1. Test Behaviour Through the Public Interface

Call the code the way its users do: the endpoint, the service function, the handler's public method. Never test private methods or internal helpers directly; if one feels like it needs its own tests, it is probably a separate module with its own public interface.

```python
# Good ✅ — goes through the endpoint a client calls
response = client.post("/items", {"category": "x"})
assert response.status_code == 201

# Bad ❌ — reaches past the interface
assert ItemService()._normalise_category("X") == "x"
```

Test behaviours, not methods: one method can have several behaviours, and one behaviour can span several methods.

### 2. Check Results, Not Calls

Assert on what the code returned, stored, or sent, not on which functions it called internally.

```python
# Good ✅ — checks the outcome
create_item(call_context=ctx, category="x")
assert Item.objects.get().category == "x"

# Bad ❌ — a change-detector test: breaks on any refactor, proves nothing works
with patch("app.services.items.save_item") as save:
    create_item(call_context=ctx, category="x")
    save.assert_called_once()
```

Asserting a call is right only when the call *is* the behaviour: an email sent, a message published, a webhook fired, an audit event emitted to an external system.

### 3. Choose Test Doubles Deliberately

Prefer, in order: **real implementation → fake → stub → mock**.

| Dependency | Use |
|---|---|
| Your own code | The real thing. Never mock internal functions or classes. |
| Your own database (a *managed* dependency) | The real test database, or an in-memory fake. Don't mock it: how your code talks to it is an implementation detail. |
| Third-party APIs, email, queues, payment providers (*unmanaged*, others can observe them) | Stub queries (canned responses); mock commands whose side effect is the behaviour. Wrap them in your own thin adapter and double that. |
| Clock, randomness, ids | Inject or freeze them. |

Put the doubles in one shared fixture (e.g. `mock_all_external_apis`) and override per test. More than ~3 `@patch` decorators on one test means the fixture is incomplete or the test is at the wrong level.

### 4. One Behaviour per Test, Named for Scenario and Outcome

The name says what happens and when. If it needs "and", split the test.

```python
# ✅
def test_permission_denied_when_user_lacks_view_permission(): ...
def test_returns_only_unresolved_items_by_default(): ...
def test_extending_with_an_empty_recruiter_clears_it(): ...

# ❌
def test_get_item(): ...
def test_status_filter(): ...
def test_validate_updates(): ...
```

File names describe scope: `test_{service_function}.py`. Split into concern files (`_permissions.py`, `_errors.py`) only past ~150 lines.

### 5. Given / When / Then

Every test has three visible parts: the starting state, the one action, the checks. Separate them with blank lines or `# Given` / `# When` / `# Then` comments. Don't interleave actions and assertions.

```python
def test_resolving_an_item_records_who_resolved_it(ctx):
    # Given
    item = ItemFactory.create(resolved=False)

    # When
    resolve_item(call_context=ctx, item_id=item.id)

    # Then
    item.refresh_from_db()
    assert item.resolved_by == ctx.user_id
```

### 6. Descriptive Over DRY

- Straight-line code: no loops, conditionals or computed expectations in a test.
- Every value the assertion depends on appears in the test itself, not hidden in a shared constant or setup.
- Factories for DB records (`ItemFactory.create(resolved=True)`), builders for external API payloads (`ApiResponseBuilder(id).paused().build()`). Both have sensible defaults; the test overrides only what it cares about.
- Never `Model.objects.create(...)` or hand-crafted API JSON from scratch in a test.
- A little duplication is fine when it makes the test easier to read.

### 7. Parametrize Variations of One Behaviour

Use `@pytest.mark.parametrize` when the same behaviour is checked with different inputs; never copy-paste a test to change one value. Give cases readable ids so a failure names the case. Don't parametrize different behaviours into one table.

```python
@pytest.mark.parametrize(
    "category", ["", "invalid", None], ids=["empty", "unknown", "missing"]
)
def test_create_rejects_unsupported_category(ctx, category):
    with pytest.raises(UnsupportedCategoryError):
        create_item(call_context=ctx, category=category)
```

### 8. Lowest Level That Catches the Bug, Once

Test each behaviour at the lowest level where it can be caught through a public interface, and don't repeat the same assertion at other levels. If a higher-level test finds a bug no lower-level test catches, add the lower-level test.

### 9. Isolated, Deterministic, Fast

- Each test runs alone and in any order; no shared mutable state between tests.
- No real network, clock or randomness; freeze or inject them.
- Fast enough to run on every change.

### 10. Specific Failures

A failure should say what broke without opening the test: one behaviour per test, readable parametrize ids, assertions on the exact value (`assert item.status == "resolved"`, not `assert item`).

### 11. Permissions and Edge Cases

- Every endpoint or service function: both the denied and the allowed path.
- Edge cases: empty results, invalid inputs (`""`, unknown, `None`), boundary values.

## Testing by Layer (Django apps)

| Layer | Owns |
|---|---|
| `models.py` | DB constraints and Pydantic validation only |
| `services/` | The use case: permissions (`PermissionDenied` for wrong and right roles), `NotFoundError` for unknown ids, the resulting state, and side effects on external systems |
| `handlers/` | All computation scenarios, through the public `compute()` |
| `http_api/` | 401 without auth, permission → status-code mapping, error response shape, one happy-path contract test per endpoint |

Service tests check the outcome of the use case through fakes or the test database. They don't assert that internal functions were called, and they don't re-test every computation the handler tests already own.

## Quick Checklist

- [ ] Does it go through a public interface, the way a user or caller would?
- [ ] Would it stay green through a refactor that keeps behaviour?
- [ ] Does it assert results or stored state, not internal calls?
- [ ] Are doubles only on unmanaged dependencies, with the DB real or faked?
- [ ] One behaviour, named for scenario and outcome?
- [ ] Given / When / Then visible?
- [ ] Factories and builders instead of raw dicts and `objects.create`, with the relevant values in the test?
- [ ] Parametrize for variations, with readable ids?
- [ ] No identical assertion already covered at another level?
- [ ] Isolated and deterministic (no real network, clock or randomness)?
- [ ] Denied and allowed paths, plus edge cases?
