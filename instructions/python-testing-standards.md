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

### Test Behavior, Not Implementation

**Good**: Test what the user experiences
```python
def test_list_returns_all_items_for_category():
    """Test listing all items for a given category."""
    item1 = ItemFactory.create()
    item2 = ItemFactory.create()

    result = list_items(call_context=context, category="some_category")

    assert len(result.items) == 2
    assert result.total == 2
```

**Bad**: Test internal implementation details
```python
def test_queryset_filter_is_called():
    """Test that queryset.filter() is called."""  # ❌ Testing implementation
    with patch('app.models.Item.objects.filter') as mock_filter:
        list_items(...)
        mock_filter.assert_called_once()
```

### Write Clear Test Descriptions

**Good**: Describe the behavior being tested
```python
def test_filter_by_status_resolved():
    """Test filtering by status='resolved' returns only resolved items."""
```

**Bad**: Describe variables or implementation
```python
def test_status_filter():
    """Test status filter."""  # ❌ Too vague
```

### Test Structure

```python
from django.test import TestCase

class ListItemsTest(TestCase):
    def setUp(self):
        """Set up test fixtures used across multiple tests."""
        self.context = build_call_context(["some.permission"])

    def test_specific_behavior(self):
        """Test description explaining what behavior is being verified."""
        item = ItemFactory.create()

        result = list_items(call_context=self.context, category="some_category")

        assert len(result.items) == 1
```

### Use Factories for Test Data

Always use factories instead of creating models directly:

```python
# Good ✅
item = ItemFactory.create(resolved=True, resolution={"outcome": "approved"})

# Bad ❌
item = Item.objects.create(category="some-category", ...)
```

### Test Permissions

Always test authorization:

```python
def test_list_without_permission():
    """Test that PermissionDenied is raised without proper permissions."""
    context = build_call_context([])  # No permissions

    with pytest.raises(PermissionDenied):
        list_items(call_context=context, category="some_category")
```

### Test Edge Cases

Cover empty results, invalid inputs, and boundary conditions:

```python
def test_list_empty_results():
    """Test that empty results are handled correctly."""
    result = list_items(call_context=self.context, category="some_category")

    assert result.items == []
    assert result.total == 0
```

## Testing by Layer

Four distinct layers, each with a different job:

| Layer | Responsibility |
|---|---|
| `models.py` | DB schema, constraints, Pydantic validation |
| `services/` | Permissions, orchestration, error handling |
| `handlers/` | Factor computation (hard business logic) |
| `http_api/` | Request/response mapping, auth, JSON:API format |

### 1. Test at the Lowest Layer Where the Behaviour Lives

If a bug can be caught at the handler layer, write it there. Never duplicate the same logical assertion at multiple layers.

### 2. Model Tests: DB Constraints and Pydantic Validation Only

Verify the database rejects invalid state and Pydantic schemas reject malformed data. Nothing else.

### 3. Service Tests: Prove Orchestration, Not Outcomes

Service tests verify:
- `PermissionDenied` raised for the right (and wrong) permissions
- `NotFoundError` for unknown IDs
- Audit logging is called with the right arguments
- Business logic function is **called** (not what it returned)

Service tests do **not** assert computed values. Mock the handler entirely.

```python
# ✅ Right — service test
mock_compute.assert_called_once_with(call_context=self.call_context)
assert result.id == item.id

# ❌ Wrong — service test asserting computed values
assert result.factors.some_field == "value"
```

### 4. Handler Tests: Own All Computation Scenarios

Test the **public** `compute()` method only — both the happy path and all edge cases. Never test private methods.

```python
# ✅ Right
result = handler.compute()
assert result.status == "paused"

# ❌ Wrong
result = handler._compute_subscription_factors()
```

### 5. HTTP Tests: Only HTTP-Specific Concerns

Test exactly four things:

| What to test | Example |
|---|---|
| Auth required | `401` without token |
| Permission → status code mapping | `403` for wrong role |
| Error response format | `{"errors": [{"code": ...}]}` |
| One happy-path contract test per endpoint | Correct shape of a `200` response |

Computation logic, filtering, and permission rules are **not** HTTP concerns.

### 6. Mock Only at External API Boundaries

Never mock internal methods. Mock only what crosses a real network boundary. Use a shared `mock_all_external_apis` conftest fixture as the baseline, then override individual mocks per scenario. A test with more than 5 `@patch` decorators signals an incomplete fixture or wrong layer.

### 7. Factories for DB Records, Builders for External API Responses

```python
# DB records → factory_boy
item = ItemFactory.create(resolved=True)

# External API JSON → builders
response = ApiResponseBuilder(id).paused().build()
```

Never manually call `Model.objects.create(...)` in tests. Never hand-craft external API response dicts from scratch. Never manually construct a domain result object just to satisfy a mock return value — that test belongs at the handler layer instead.

### 8. Parametrize for Variations

When the same behaviour needs to be verified with different inputs, use `@pytest.mark.parametrize`. Never copy-paste a test to change one variable.

```python
@pytest.mark.parametrize("category", ["", "invalid", None])
def test_create_with_invalid_category_raises(call_context, category):
    with pytest.raises(UnsupportedCategoryError):
        create(call_context=call_context, category=category, ...)
```

### 9. Test Names Describe Behaviour; File Names Describe Scope

```python
# ✅
def test_permission_denied_when_user_has_wrong_category_view_permission():
def test_returns_only_unresolved_items_by_default():
def test_compute_returns_none_for_active_subscription():

# ❌
def test_get_item():
def test_list_filter():
def test_subscription_factors():
```

File names follow `test_{service_function}.py`. Split into concern-specific files (`_permissions.py`, `_error_handling.py`) only when the primary file exceeds ~150 lines.

## Quick Checklist

Before committing a test, verify:

- [ ] Is it testing the lowest layer where the behaviour lives?
- [ ] If it is a service test, does it avoid asserting computed values?
- [ ] Does it mock only external APIs, never internal methods?
- [ ] If I added more than 3 `@patch` decorators, did I extend the shared fixture instead?
- [ ] Is it using a factory (DB) or builder (external API) instead of raw dicts or domain objects?
- [ ] Is it `pytest` style with plain `assert`?
- [ ] Could multiple similar tests be collapsed into one `@pytest.mark.parametrize`?
- [ ] Does the test name describe the scenario and expected outcome?
- [ ] Is there no identical assertion already covered in another layer's tests?
