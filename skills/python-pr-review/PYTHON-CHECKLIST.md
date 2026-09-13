# Python Logic & Code-Smell Checklist

The contract for `@review-python-logic`. For every finding: cite `file:line`, explain **why** it matters, and give a concrete fix. If a check doesn't apply to the diff, skip it silently.

**Convention authority**: personal standards win over repo conventions. F-strings for all log messages (per `~/.config/opencode/instructions/logging-standards.md`).

## 1. Correctness traps

- [ ] Mutable default argument (`def f(items=[])`, `={}`) → `None` sentinel, init inside the body.
- [ ] Mutable class attribute shared across instances → move into `__init__`.
- [ ] `== None` / `== True` / `== False` → `is None` / plain truthiness.
- [ ] `is` used for value comparison (`x is "str"`, `x is 1`) → `==`.
- [ ] `assert` used for runtime/data validation — stripped under `python -O` → raise a real exception.
- [ ] Bare `except:` or `except Exception:` that swallows (no log, no re-raise) → catch the narrowest exception; log with context or re-raise.
- [ ] `try` body wraps more than the call that can fail → shrink to the risky line.
- [ ] `dict.get()` masking a key that MUST exist → `[]` + `KeyError`, or explicit check with a clear error message.
- [ ] Inconsistent return shapes from one function (sometimes `None`, sometimes a dict; sometimes list, sometimes item) → one return contract; raise or use a result type.
- [ ] Boolean-trap parameters (`f(x, True, False)` unreadable at the call site) → keyword-only args, an enum, or split functions.
- [ ] Returning internal mutable state (`self._cache`, a class-level list) → return a copy; aliasing bugs.
- [ ] Late binding in closures/lambdas defined in a loop → bind as default arg (`lambda x=x: ...`) or `functools.partial`.
- [ ] Shadowing builtins (`id`, `type`, `list`, `dict`, `input`, `filter`) → rename.
- [ ] Float equality (`==`) or float for money → `math.isclose` / `Decimal` for currency.
- [ ] Naive datetimes (`datetime.now()`, `datetime.utcnow()` — deprecated) → `datetime.now(timezone.utc)`; ISO 8601 UTC at boundaries.
- [ ] `time.time()` for measuring durations → `time.monotonic()`.
- [ ] Shallow-copy misuse on nested structures → `copy.deepcopy` where mutation would leak.
- [ ] String building in a loop (`s += ...`) → `"".join(parts)`.
- [ ] Unvalidated keyed access on user/external data deep in the call stack → validate once at the boundary, then trust internally.

## 2. Resources & I/O

- [ ] Files / sockets / connections opened without `with` → context managers everywhere.
- [ ] Network call without an explicit `timeout=` → every outbound call gets one.
- [ ] Hand-rolled retry loops, or retries without backoff/jitter → tenacity-style backoff, capped attempts.
- [ ] Unbounded reads: `f.read()` on large/unknown files, `resp.json()` on unbounded payloads, fetching ALL rows → stream, paginate, or cap.

## 3. Errors & logging

- [ ] Sentinels (`return None` / `return -1`) for error conditions → raise domain exceptions; reserve `None` for "legitimately absent".
- [ ] Exceptions used for routine branch logic the caller must handle → return values for expected cases.
- [ ] `logger.error` inside an exception handler that doesn't re-raise → `logger.exception` (keeps the traceback).
- [ ] Log AND re-raise at the same layer → pick one; double-logging spams.
- [ ] Log messages without context (no ids / values / operation name) → include entity ids, counts, operation.
- [ ] Log messages using `%` formatting or `.format()` → always f-strings (personal logging standard).
- [ ] Third-party exceptions leaking across layer boundaries → wrap in domain exceptions at the seam.
- [ ] Stack traces / internal error text returned to API clients → generic message + machine-readable `code`; details logged.

## 4. Idioms & readability

- [ ] Index loops (`for i in range(len(xs))`) → `enumerate`, `zip`, `.items()`.
- [ ] Comprehension nested more than one level, or with side effects → plain loop or named helper.
- [ ] Nesting deeper than ~3 levels → guard clauses / early returns / extraction.
- [ ] Function doing several things, or very long → extract; name after the domain concept.
- [ ] More than ~4–5 positional parameters → params object / dataclass / keyword-only.
- [ ] Magic numbers/strings scattered → module constants or `Enum`/`StrEnum`.
- [ ] Stringly-typed state (`status == "actve"` typo risk) → `Enum` / `Literal`.
- [ ] `import *`, unused imports, or function-level imports hiding circular dependencies → explicit imports; fix the cycle, don't hide it.
- [ ] Re-implementing the stdlib (`itertools`, `functools`, `collections`, `dataclasses`, `pathlib`) → use it.
- [ ] `os.path` string joins → `pathlib.Path`.
- [ ] `print(`, `pdb`, `breakpoint()`, commented-out code, TODO/FIXME without owner + ticket → remove or ticket it.
- [ ] Stale comments contradicting the code, or comments narrating the obvious → delete or fix; comments explain WHY.

## 5. Typing & Pydantic — CONDITIONAL

Apply this section **only to files that already contain type hints or import pydantic**. Never flag the *absence* of types and never demand a types/pydantic retrofit in an untyped file. When hints exist, review their correctness rigorously:

- [ ] `Any` creeping into public interfaces → precise types.
- [ ] `cast()` or `# type: ignore` without a stated reason → justify or fix the underlying issue.
- [ ] Wrong `Optional` (`x: int = None` without `| None`); missing return types on public functions in an otherwise-typed file.
- [ ] Pydantic v1/v2 API mixed (`.dict()` / `.json()` vs `model_dump()` / `model_dump_json()`) → match the codebase's major version.
- [ ] Validation re-run at every layer instead of once at the boundary → validate at ingress, pass typed objects inward.
- [ ] Hand-rolled dict shapes at boundaries where the codebase uses pydantic models.

## 6. Security & data protection

- [ ] SQL/query built with f-strings or `%` formatting → parameterized queries / ORM.
- [ ] Secrets, tokens, passwords in code, fixtures, or logs → secret store; scrub logs.
- [ ] PII/PHI (names, emails, DOB, clinical/financial data) in logs → GDPR: log ids, not payloads.
- [ ] `pickle.loads` / `yaml.load` on untrusted data → JSON / `yaml.safe_load`.
- [ ] User input used in file paths → sanitize and confine to a base dir (`Path.resolve()` + prefix check).
- [ ] User-supplied URLs fetched server-side → SSRF guard (allowlist, block internal ranges).
- [ ] New endpoint without object-level authorization (IDOR) → per-object ownership/permission check, not just "logged in".
- [ ] `subprocess` with `shell=True`, `eval`, `exec` → avoid; if truly needed, strict input validation.
- [ ] Tokens/keys generated with `random` → `secrets` module.
- [ ] New dependency: pinned? maintained? license acceptable?

## 7. Performance

- [ ] ORM/DB calls inside loops or comprehensions (N+1) → bulk fetch / `select_related` / `prefetch_related` / `WHERE id IN (...)`.
- [ ] Unbounded querysets returned to callers → paginate; never `.all()` on a growing table.
- [ ] Blocking I/O (`requests`, `time.sleep`, sync DB driver) inside `async def` → async client or `asyncio.to_thread`.
- [ ] Whole resultsets/files loaded into memory where streaming works → generators.
- [ ] Expensive recomputation in a `property` called in loops → compute once / cache.
- [ ] Cache without eviction, or cache keys missing tenant/user scoping → bounded cache, scoped keys.
- [ ] O(n²) membership checks (`x in some_list` inside a loop) → `set`.

## 8. Concurrency

- [ ] Read-modify-write on shared rows without locking/atomic ops → atomic `UPDATE`, DB constraints, `select_for_update`.
- [ ] Shared mutable module-level state mutated per request → request-scoped objects.
- [ ] Coroutine created but never awaited; `asyncio.create_task` fire-and-forget with no error handling → await, or supervised task with exception logging.
- [ ] Thread-unsafe singleton / lazy init without a lock → lock or eager init.
