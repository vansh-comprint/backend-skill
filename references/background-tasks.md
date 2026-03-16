# Background Tasks Reference

## Decision Table

FastAPI offers two approaches to background work: the built-in `BackgroundTasks` for lightweight operations and Celery for heavy, distributed, or retriable work.

| Scenario                    | Use              | Reason                              |
|-----------------------------|------------------|-------------------------------------|
| Send email after signup     | `BackgroundTasks` | Quick, no external dependencies     |
| Log analytics event         | `BackgroundTasks` | Fire-and-forget, fast               |
| Process uploaded file       | Celery           | Could take minutes                  |
| Generate PDF report         | Celery           | CPU-intensive                       |
| Sync data with third party  | Celery           | Needs retries, error handling       |
| Scheduled/periodic jobs     | Celery Beat      | Cron-like scheduling                |

The dividing line: if the task completes in under 30 seconds, does not need retries, and does not need to survive a server restart, use `BackgroundTasks`. For everything else, use Celery.

### BackgroundTasks Limitations

- Runs in the same process as the web server. If the server restarts, the task is lost.
- No retry mechanism. If the task fails, it fails silently.
- No monitoring. There is no way to check whether a task succeeded or how long it took.
- No distribution. The task runs on the instance that received the HTTP request.
- Synchronous task functions block the event loop. Always use async functions.

### Celery Advantages

- Tasks are persisted in a message broker (Redis). They survive server restarts.
- Built-in retry with configurable backoff and max attempts.
- Task status tracking (PENDING, STARTED, SUCCESS, FAILURE).
- Horizontal scaling -- add more worker processes to increase throughput.
- Celery Beat provides cron-like scheduling for periodic tasks.

---

## BackgroundTasks Usage

FastAPI's `BackgroundTasks` is a dependency that queues functions to run after the response is sent to the client.

### Basic Pattern

```python
from fastapi import APIRouter, BackgroundTasks
from app.api.dependencies import UserServiceDep
from app.schemas.common import StandardResponse
from app.utils.logger import setup_logger

router = APIRouter()
logger = setup_logger("tasks")


async def send_welcome_email(email: str, name: str):
    """Runs after response is sent to client."""
    logger.info("Sending welcome email", extra={"email": email})
    # await email_service.send(to=email, template="welcome", name=name)


@router.post("/register", response_model=StandardResponse, status_code=201)
async def register(
    data: RegisterRequest,
    service: UserServiceDep,
    background_tasks: BackgroundTasks,
):
    user = await service.create_user(data)

    # Queued -- runs AFTER response is returned
    background_tasks.add_task(send_welcome_email, user.email, user.full_name)

    return StandardResponse(code=201, data=user, message="Registration successful")
```

The `background_tasks.add_task()` call does not execute the function immediately. It queues it to run after the response is sent. The client receives the 201 response without waiting for the email to be sent.

### Multiple Tasks

Multiple tasks can be queued in a single endpoint. They run sequentially in the order they were added.

```python
@router.post("/orders", response_model=StandardResponse, status_code=201)
async def create_order(
    data: OrderCreate,
    service: OrderServiceDep,
    background_tasks: BackgroundTasks,
):
    order = await service.create_order(data)

    # All run sequentially after response
    background_tasks.add_task(send_order_confirmation, order.id)
    background_tasks.add_task(update_inventory, order.items)
    background_tasks.add_task(notify_warehouse, order.id)

    return StandardResponse(code=201, data=order, message="Order placed")
```

### Dependency Shorthand

Use an annotated type alias to keep endpoint signatures clean:

```python
from typing import Annotated
from fastapi import BackgroundTasks, Depends

BgTasks = Annotated[BackgroundTasks, Depends()]


@router.post("/users")
async def create_user(data: UserCreate, service: UserServiceDep, bg: BgTasks):
    user = await service.create_user(data)
    bg.add_task(send_welcome_email, user.email, user.full_name)
    return StandardResponse(code=201, data=user)
```

---

## Celery Configuration

Celery uses Redis as both the message broker (task queue) and the result backend (task status storage).

```bash
poetry add celery[redis]
```

```python
# app/worker/celery_app.py
from celery import Celery
from app.config.settings import settings

celery_app = Celery(
    "worker",
    broker=settings.REDIS_URL,
    backend=settings.REDIS_URL,
)

celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
    task_track_started=True,
    task_acks_late=True,           # retry if worker dies mid-task
    worker_prefetch_multiplier=1,  # fair scheduling
    result_expires=3600,           # results expire in 1 hour
)

# Auto-discover tasks in all modules
celery_app.autodiscover_tasks(["app.worker.tasks"])
```

### Configuration Explained

- **`task_serializer="json"`** -- Tasks and results are serialized as JSON. This is safer than binary serialization formats and makes task payloads inspectable in monitoring tools.
- **`task_track_started=True`** -- The broker records when a task transitions from PENDING to STARTED. Without this, tasks jump directly from PENDING to SUCCESS/FAILURE.
- **`task_acks_late=True`** -- The worker acknowledges the task only after it completes, not when it starts. If the worker crashes mid-task, the broker redelivers the task to another worker.
- **`worker_prefetch_multiplier=1`** -- Each worker fetches one task at a time. This prevents a single worker from hoarding tasks while others sit idle. Important for tasks with variable execution times.
- **`result_expires=3600`** -- Task results are deleted from Redis after one hour. This prevents unbounded memory growth. Adjust based on how long clients need to poll for results.
- **`autodiscover_tasks`** -- Celery scans the specified module paths for functions decorated with `@celery_app.task` and registers them automatically.

### Project Structure

```
app/
├── worker/
│   ├── __init__.py
│   ├── celery_app.py       # Celery configuration
│   └── tasks.py            # Task definitions
```

---

## Task Definitions

Tasks are regular Python functions decorated with `@celery_app.task`. Use `bind=True` to access the task instance for retry logic.

```python
# app/worker/tasks.py
from app.worker.celery_app import celery_app
import logging

logger = logging.getLogger("celery.tasks")


@celery_app.task(bind=True, max_retries=3, default_retry_delay=60)
def send_email_task(self, to: str, subject: str, body: str):
    """Send email with automatic retries."""
    try:
        logger.info("Sending email to %s", to)
        # email_client.send(to=to, subject=subject, body=body)
    except Exception as exc:
        logger.error("Email failed, retrying: %s", str(exc))
        raise self.retry(exc=exc)


@celery_app.task(bind=True, max_retries=2)
def process_file_task(self, file_path: str, user_id: int):
    """Process uploaded file (CPU-intensive)."""
    try:
        logger.info("Processing file %s for user %d", file_path, user_id)
        # result = heavy_processing(file_path)
        # store_result(user_id, result)
    except Exception as exc:
        raise self.retry(exc=exc)


@celery_app.task
def generate_report_task(report_type: str, params: dict):
    """Generate PDF report."""
    logger.info("Generating %s report", report_type)
    # pdf = report_generator.create(report_type, **params)
    # storage.upload(pdf)
```

### Retry Behavior

- **`bind=True`** -- Passes the task instance as `self`, giving access to `self.retry()`.
- **`max_retries=3`** -- The task is retried up to 3 times. After the final retry fails, the task is marked as FAILURE.
- **`default_retry_delay=60`** -- Wait 60 seconds between retries. This gives transient failures (network timeouts, rate limits) time to resolve.
- **`self.retry(exc=exc)`** -- Re-raises the exception to trigger the retry mechanism. The original exception is preserved in the task result for debugging.

For exponential backoff, calculate the delay dynamically:

```python
@celery_app.task(bind=True, max_retries=5)
def resilient_task(self, data: dict):
    try:
        # ... work ...
        pass
    except Exception as exc:
        delay = 2 ** self.request.retries * 30  # 30s, 60s, 120s, 240s, 480s
        raise self.retry(exc=exc, countdown=delay)
```

### Dispatching Tasks from Endpoints

Use `.delay()` or `.apply_async()` to send tasks to the Celery worker. Both return immediately with an `AsyncResult` object.

```python
# api/v1/endpoints/reports.py
from fastapi import APIRouter
from app.worker.tasks import generate_report_task
from app.schemas.common import StandardResponse

router = APIRouter()


@router.post("/reports/generate", response_model=StandardResponse)
async def request_report(data: ReportRequest, user: CurrentUser):
    # Dispatch to Celery -- returns immediately
    task = generate_report_task.delay(
        report_type=data.report_type,
        params=data.params,
    )

    return StandardResponse(
        data={"task_id": task.id},
        message="Report generation started",
    )
```

---

## Task Status Endpoints

For long-running tasks, provide a status endpoint so clients can poll for completion.

```python
@router.get("/reports/status/{task_id}", response_model=StandardResponse)
async def check_report_status(task_id: str):
    from celery.result import AsyncResult
    result = AsyncResult(task_id)

    return StandardResponse(data={
        "task_id": task_id,
        "status": result.status,        # PENDING, STARTED, SUCCESS, FAILURE
        "result": result.result if result.ready() else None,
    })
```

### Status Values

| Status    | Meaning                                       |
|-----------|-----------------------------------------------|
| `PENDING` | Task is in the queue, not yet picked up       |
| `STARTED` | Worker has begun executing the task            |
| `SUCCESS` | Task completed successfully                   |
| `FAILURE` | Task raised an unhandled exception             |
| `RETRY`   | Task failed and is waiting to be retried       |

The `result.result` field contains the task's return value on SUCCESS or the exception instance on FAILURE. Only access it after checking `result.ready()` to avoid blocking.

### Client Polling Pattern

Clients should poll the status endpoint at intervals (e.g., every 2-5 seconds) until the status is `SUCCESS` or `FAILURE`. For real-time updates, consider WebSockets or Server-Sent Events instead of polling.

---

## Celery Beat

Celery Beat is a scheduler that dispatches tasks at defined intervals. It runs as a separate process alongside the worker.

```python
# app/worker/celery_app.py (add to configuration)
from celery.schedules import crontab

celery_app.conf.beat_schedule = {
    "cleanup-expired-tokens": {
        "task": "app.worker.tasks.cleanup_expired_tokens",
        "schedule": crontab(hour=2, minute=0),  # daily at 2 AM UTC
    },
    "send-daily-digest": {
        "task": "app.worker.tasks.send_daily_digest",
        "schedule": crontab(hour=8, minute=0),   # daily at 8 AM UTC
    },
    "sync-external-data": {
        "task": "app.worker.tasks.sync_external_data",
        "schedule": 300.0,  # every 5 minutes (in seconds)
    },
}
```

### Schedule Types

- **`crontab(hour=2, minute=0)`** -- Standard cron syntax. Supports `minute`, `hour`, `day_of_week`, `day_of_month`, and `month_of_year` fields.
- **`300.0`** -- Interval in seconds. The task runs every 300 seconds (5 minutes) regardless of when the Beat process started.
- **`crontab(minute="*/15")`** -- Every 15 minutes. Standard cron wildcards work.
- **`crontab(hour=9, minute=0, day_of_week="mon")`** -- Every Monday at 9 AM UTC.

### Running Beat

```bash
# Start the Beat scheduler
poetry run celery -A app.worker.celery_app beat --loglevel=info

# Start a worker to process the scheduled tasks
poetry run celery -A app.worker.celery_app worker --loglevel=info
```

Both processes must be running. Beat dispatches tasks to the broker; the worker picks them up and executes them.

---

## Docker Compose

In production, the FastAPI app, Celery worker, and Celery Beat run as separate services. They share the same codebase (Docker image) but run different commands.

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports: ["8000:8000"]
    depends_on: [db, redis]
    environment:
      DATABASE_URL: postgresql+asyncpg://user:pass@db/appdb
      REDIS_URL: redis://redis:6379

  celery-worker:
    build: .
    command: celery -A app.worker.celery_app worker --loglevel=info
    depends_on: [db, redis]
    environment:
      DATABASE_URL: postgresql+asyncpg://user:pass@db/appdb
      REDIS_URL: redis://redis:6379

  celery-beat:
    build: .
    command: celery -A app.worker.celery_app beat --loglevel=info
    depends_on: [redis]
    environment:
      REDIS_URL: redis://redis:6379

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: appdb

  redis:
    image: redis:7-alpine
```

### Service Roles

- **`app`** -- The FastAPI web server. Handles HTTP requests and dispatches Celery tasks via `.delay()`.
- **`celery-worker`** -- Processes tasks from the Redis queue. Needs access to both the database (for tasks that read/write data) and Redis (for receiving tasks and storing results). Scale horizontally by running multiple worker instances.
- **`celery-beat`** -- Runs the scheduler. Only one Beat instance should run at a time to avoid duplicate task dispatches. It needs Redis access to push scheduled tasks to the broker. It does not need database access unless the schedule itself is stored in a database.
- **`db`** -- PostgreSQL database. Used by the app and worker services.
- **`redis`** -- Message broker and result backend. Used by all Celery components.

### Scaling Workers

To run multiple worker instances:

```yaml
  celery-worker:
    build: .
    command: celery -A app.worker.celery_app worker --loglevel=info --concurrency=4
    deploy:
      replicas: 2
```

This runs 2 worker containers, each with 4 concurrent worker threads, for a total of 8 concurrent task slots. Adjust `concurrency` and `replicas` based on the workload profile and available resources.
