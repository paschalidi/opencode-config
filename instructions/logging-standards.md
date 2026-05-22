# Logging Standards

## F-strings for log messages

Always use f-strings for log message formatting. Do not use `%` formatting or `.format()`.

```python
# Good
logger.info(f"Step '{step_name}' done", extra=log_extra)

# Bad
logger.info("Step '%s' done", step_name, extra=log_extra)
logger.info("Step '{}' done".format(step_name), extra=log_extra)
```

This applies to all log levels: `logger.debug`, `logger.info`, `logger.warning`, `logger.error`, `logger.exception`, `logger.critical`.

## Why

- F-strings are evaluated immediately, making the code more readable
- Consistent with modern Python style (PEP 498)
- Avoids confusion with the `extra` parameter when using `%` formatting
