# FastAPI Application Scaffold

Planned application structure:

```text
app/
|-- __init__.py
|-- main.py
|-- api/
|   |-- __init__.py
|   `-- v1/
|       |-- __init__.py
|       |-- router.py
|       `-- endpoints/
|           |-- __init__.py
|           |-- health.py
|           `-- payments.py
|-- core/
|   |-- __init__.py
|   |-- config.py
|   `-- logging.py
|-- db/
|   |-- __init__.py
|   |-- base.py
|   `-- session.py
|-- models/
|   |-- __init__.py
|   |-- idempotency_key.py
|   |-- ledger_entry.py
|   |-- payment_event.py
|   `-- payment_intent.py
|-- schemas/
|   |-- __init__.py
|   `-- payment.py
|-- services/
|   |-- __init__.py
|   |-- idempotency.py
|   |-- ledger.py
|   `-- payments.py
`-- workers/
    |-- __init__.py
    `-- payment_processor.py
```

## `main.py`

```python
from fastapi import FastAPI

from app.api.v1.router import api_router
from app.core.config import settings


def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.PROJECT_NAME,
        version=settings.VERSION,
    )

    app.include_router(api_router, prefix="/api/v1")

    return app


app = create_app()
```

## `api/v1/router.py`

```python
from fastapi import APIRouter

from app.api.v1.endpoints import health, payments


api_router = APIRouter()
api_router.include_router(health.router, tags=["health"])
api_router.include_router(payments.router, prefix="/payments", tags=["payments"])
```

## `api/v1/endpoints/health.py`

```python
from fastapi import APIRouter


router = APIRouter()


@router.get("/health-check")
def health_check() -> dict[str, str]:
    return {"status": "ok"}
```

## `api/v1/endpoints/payments.py`

```python
from fastapi import APIRouter


router = APIRouter()


# POST /payments
# GET /payments/{payment_id}
# POST /payments/{payment_id}/capture
```

## `core/config.py`

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    PROJECT_NAME: str = "Payment Orchestration Platform"
    VERSION: str = "0.1.0"
    DATABASE_URL: str
    AWS_REGION: str = "us-east-1"
    SQS_PAYMENT_QUEUE_URL: str | None = None


settings = Settings()
```

## `db/session.py`

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from app.core.config import settings


engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

## `db/base.py`

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```
