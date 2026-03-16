# Logging Reference

## Logger Setup

All logging uses `python-json-logger` to produce structured JSON output in non-debug environments. A single factory function creates loggers throughout the application.

```python
# utils/logger.py
import logging
import sys
from pythonjsonlogger import jsonlogger
from app.config.settings import settings


def setup_logger(name: str = "app") -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(getattr(logging, settings.LOG_LEVEL.upper()))

    if logger.handlers:
        return logger

    handler = logging.StreamHandler(sys.stdout)

    if settings.DEBUG:
        formatter = logging.Formatter(
            "%(asctime)s | %(levelname)-8s | %(name)s | %(message)s"
        )
    else:
        formatter = jsonlogger.JsonFormatter(
            fmt="%(asctime)s %(levelname)s %(name)s %(message)s",
            rename_fields={"asctime": "timestamp", "levelname": "level"},
        )

    handler.setFormatter(formatter)
    logger.addHandler(handler)
    return logger


logger = setup_logger()
```

Install the dependency:

```bash
poetry add python-json-logger
```

The dual-format approach keeps local development readable while producing machine-parseable JSON in staging and production. The `setup_logger` function is idempotent -- calling it twice with the same name returns the existing logger without adding duplicate handlers. The handler guard (`if logger.handlers: return logger`) prevents the common problem of duplicate log lines that occurs when multiple modules import and call the setup function.

### How It Works

The function accepts a `name` parameter that creates a hierarchical logger. Calling `setup_logger("services.user")` creates a logger under the `services` namespace. Python's logging module uses dot-separated names to form a hierarchy: a message logged to `services.user` propagates up to `services` and then to the root logger. This means you can configure log levels at any level of the hierarchy. For example, setting `services` to `WARNING` silences info-level messages from all service loggers at once.

The `JsonFormatter` from `python-json-logger` serializes the standard `LogRecord` fields plus anything passed in the `extra` dict into a single JSON line. The `rename_fields` parameter maps Python's default field names (`asctime`, `levelname`) to the names expected by log aggregation platforms (`timestamp`, `level`). Any additional keys in `extra` -- such as `request_id`, `user_id`, or `email` -- are included as top-level fields in the JSON output.

### Environment-Specific Levels

| Environment | LOG_LEVEL | Format         |
|-------------|-----------|----------------|
| Development | DEBUG     | Human-readable |
| Staging     | INFO      | JSON           |
| Production  | WARNING   | JSON           |

Configure `LOG_LEVEL` and `DEBUG` via environment variables in `settings.py`. Development uses the plain-text formatter so stack traces and debug messages are easy to read in the terminal. Staging and production use the JSON formatter so logs can be ingested by centralized logging systems.

---

## Request ID Middleware

Every inbound request gets a unique identifier. If the client sends an `X-Request-ID` header, that value is reused; otherwise, a UUID is generated. The ID is stored in a `ContextVar` so any code running within that request can retrieve it without passing it through function arguments.

```python
# core/middleware.py
import uuid
from contextvars import ContextVar
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi import Request

request_id_var: ContextVar[str] = ContextVar("request_id", default="")


class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        req_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
        request_id_var.set(req_id)
        request.state.request_id = req_id

        response = await call_next(request)
        response.headers["X-Request-ID"] = req_id
        return response


def get_request_id() -> str:
    return request_id_var.get()
```

Key details:

- **ContextVar propagation** -- `ContextVar` is coroutine-safe. Each concurrent request gets its own value automatically. No thread-local hacks required. When an `async def` endpoint handler is awaited, the `ContextVar` value is preserved across `await` boundaries within the same request.
- **Dual storage** -- The request ID is stored in both the `ContextVar` (for access from anywhere in the call stack) and `request.state` (for access within middleware that has the `Request` object). This redundancy simplifies access patterns: middleware reads from `request.state`, services call `get_request_id()`.
- **Response header** -- The same ID is returned in the `X-Request-ID` response header. Clients and frontend developers can use it to correlate requests with backend logs. Support teams can ask users to provide the request ID when reporting issues, then search logs by that ID to reconstruct what happened.
- **Client-provided IDs** -- If the client sends an `X-Request-ID` header, that value is reused. This is important for distributed systems where an API gateway or load balancer generates the ID upstream. Reusing it preserves traceability across the entire request chain.
- **`get_request_id()` helper** -- Services and utilities import this function to attach the current request ID to their log entries without needing access to the `Request` object. This decouples the logging concern from the HTTP layer.

---

## Request/Response Logging Middleware

A second middleware layer records the HTTP method, path, status code, and duration of every request. It uses `time.perf_counter()` for high-resolution timing.

```python
# core/middleware.py (continued)
import time
from app.utils.logger import setup_logger

access_logger = setup_logger("access")


class RequestLoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start = time.perf_counter()

        access_logger.info(
            "Request started",
            extra={
                "request_id": getattr(request.state, "request_id", ""),
                "method": request.method,
                "path": request.url.path,
                "client_ip": request.client.host if request.client else "",
            },
        )

        response = await call_next(request)
        duration_ms = (time.perf_counter() - start) * 1000

        access_logger.info(
            "Request completed",
            extra={
                "request_id": getattr(request.state, "request_id", ""),
                "method": request.method,
                "path": request.url.path,
                "status_code": response.status_code,
                "duration_ms": round(duration_ms, 2),
            },
        )
        return response
```

### Middleware Registration Order

Starlette processes middleware in reverse registration order. Register the logging middleware first, then the request ID middleware, so the ID is available when logging runs.

```python
# main.py
app.add_middleware(RequestLoggingMiddleware)  # logs timing
app.add_middleware(RequestIDMiddleware)        # sets ID first (runs before logging)
```

`RequestIDMiddleware` is registered second but executes first. By the time `RequestLoggingMiddleware.dispatch` fires, `request.state.request_id` is already set.

### What Gets Logged

Each request produces two log entries:

1. **Request started** -- method, path, client IP, request ID. Emitted before the route handler runs.
2. **Request completed** -- method, path, status code, duration in milliseconds, request ID. Emitted after the response is produced.

The `duration_ms` field measures wall-clock time from the moment the request enters the logging middleware to the moment the response is ready. This includes route handler execution, database queries, and any downstream service calls.

---

## Service-Level Logging

Services create their own named loggers and attach the request ID from the `ContextVar`. This ties business logic logs to the same request trace.

```python
# services/UserService.py
from app.utils.logger import setup_logger
from app.core.middleware import get_request_id

logger = setup_logger("services.user")


class UserService:
    def __init__(self, repository: UserRepositoryProtocol):
        self.repository = repository

    async def create_user(self, data):
        logger.info("Creating user", extra={
            "request_id": get_request_id(), "email": data.email,
        })

        if await self.repository.get_by_email(data.email):
            logger.warning("Duplicate email", extra={
                "request_id": get_request_id(), "email": data.email,
            })
            raise ConflictError("User", data.email)

        user = await self.repository.create(data.model_dump())
        logger.info("User created", extra={
            "request_id": get_request_id(), "user_id": user.id,
        })
        return user
```

### Naming Convention

Use dotted names that reflect the module path: `services.user`, `services.order`, `repositories.user`. This makes it easy to filter logs by component in centralized logging dashboards.

### Pattern

Every service method should log:

1. **Entry** -- what operation is starting, with identifying context (email, ID, etc.).
2. **Branches** -- warnings for expected-but-notable conditions (duplicate email, missing resource).
3. **Completion** -- what was produced, with identifiers for the created/updated resource.

Always include `request_id` in the `extra` dict so every log line from a single request can be correlated. The `get_request_id()` function reads from the `ContextVar` set by `RequestIDMiddleware`, so it works anywhere in the call stack without passing the request object through every layer.

### get_logger() Convenience

For services that log frequently, a convenience wrapper can reduce boilerplate by automatically attaching the request ID:

```python
# utils/logger.py (addition)
def get_logger(name: str) -> logging.LoggerAdapter:
    base_logger = setup_logger(name)

    class RequestContextAdapter(logging.LoggerAdapter):
        def process(self, msg, kwargs):
            extra = kwargs.get("extra", {})
            extra["request_id"] = get_request_id()
            kwargs["extra"] = extra
            return msg, kwargs

    return RequestContextAdapter(base_logger, {})
```

With this adapter, service code becomes cleaner:

```python
from app.utils.logger import get_logger

logger = get_logger("services.order")


class OrderService:
    async def create_order(self, data):
        logger.info("Creating order", extra={"items": len(data.items)})
        # request_id is automatically included
```

The adapter intercepts every log call and injects the `request_id` from the current `ContextVar`. Services no longer need to call `get_request_id()` manually in every log statement.

---

## JSON Output Compatibility

In production, every log line is a single JSON object:

```json
{
  "timestamp": "2025-01-15T09:30:45.123Z",
  "level": "INFO",
  "name": "access",
  "message": "Request completed",
  "request_id": "a1b2c3d4-e5f6-7890",
  "method": "POST",
  "path": "/api/v1/users",
  "status_code": 201,
  "duration_ms": 45.23
}
```

This format is directly parseable by the major log aggregation platforms:

- **ELK Stack (Elasticsearch + Logstash + Kibana)** -- Logstash ingests JSON lines natively. Fields like `request_id`, `method`, and `status_code` become searchable and filterable in Kibana without additional parsing rules.
- **Datadog** -- The Datadog agent reads JSON logs from stdout. The `timestamp` field is recognized automatically. Custom facets can be created from `request_id`, `duration_ms`, and any other fields in `extra`.
- **AWS CloudWatch** -- CloudWatch Logs Insights queries JSON fields directly. For example: `fields @timestamp, method, path, duration_ms | filter status_code >= 500 | sort duration_ms desc`.
- **Grafana Loki** -- Loki stores log lines and indexes labels. JSON fields can be extracted at query time using LogQL's `json` parser: `{app="myservice"} | json | status_code >= 400`.

The key advantage of structured JSON logging is that no regex-based parsing is needed on the aggregation side. Every field is typed and accessible from the moment the log line is written. This eliminates parsing failures, reduces ingestion latency, and makes it possible to build dashboards and alerts based on exact field values (e.g., alert when `duration_ms > 5000` for any request, grouped by `path`).

### Correlation Across Services

If the application calls downstream services, forward the `X-Request-ID` header in outbound HTTP requests:

```python
import httpx
from app.core.middleware import get_request_id


async def call_payment_service(order_id: int, amount: float) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://payments.internal/charge",
            json={"order_id": order_id, "amount": amount},
            headers={"X-Request-ID": get_request_id()},
        )
        return response.json()
```

The same request ID will appear in logs from both services, enabling end-to-end request tracing across a microservices architecture without a full distributed tracing system like Jaeger or Zipkin. When debugging a failed payment, searching for the request ID returns log entries from the API server, the payment service, and any other downstream services that propagated the header.

### Log Levels in Practice

Use log levels consistently across all services:

| Level     | Usage                                                                 |
|-----------|-----------------------------------------------------------------------|
| `DEBUG`   | Detailed diagnostic information. Variable values, query parameters.    |
| `INFO`    | Normal operations. Request started/completed, resource created.        |
| `WARNING` | Expected-but-notable conditions. Duplicate email, rate limit close.    |
| `ERROR`   | Unexpected failures. Database connection lost, external API timeout.   |
| `CRITICAL`| System-level failures. Application cannot start, data corruption.      |

In production with `LOG_LEVEL=WARNING`, only `WARNING`, `ERROR`, and `CRITICAL` messages are emitted. This keeps log volume manageable while still capturing issues. In staging with `LOG_LEVEL=INFO`, normal operation logs are included for debugging integration issues. In development with `LOG_LEVEL=DEBUG`, everything is logged for maximum visibility during feature development.
