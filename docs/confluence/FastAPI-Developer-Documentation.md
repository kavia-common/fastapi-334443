# FastAPI Developer Documentation

> **Version:** 0.135.1  
> **License:** MIT  
> **Author:** Sebastián Ramírez (tiangolo@gmail.com)  
> **Homepage:** [https://github.com/fastapi/fastapi](https://github.com/fastapi/fastapi)  
> **Official Docs:** [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)

---

## Table of Contents

1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Getting Started](#getting-started)
4. [Core Concepts](#core-concepts)
   - [FastAPI Application](#fastapi-application)
   - [Routing and Path Operations](#routing-and-path-operations)
   - [Request Parameters](#request-parameters)
   - [Dependency Injection](#dependency-injection)
   - [Security](#security)
   - [Middleware](#middleware)
   - [OpenAPI and Documentation](#openapi-and-documentation)
5. [CLI Usage](#cli-usage)
6. [Configuration and Environment](#configuration-and-environment)
7. [Testing](#testing)
8. [Development Workflow](#development-workflow)
9. [API Reference Summary](#api-reference-summary)
10. [Troubleshooting](#troubleshooting)

---

## Overview

FastAPI is a modern, high-performance Python web framework for building APIs. It is built on top of [Starlette](https://www.starlette.io/) for the web layer and [Pydantic](https://docs.pydantic.dev/) for data validation and serialization using Python type hints.

### Key Features

- **High Performance:** On par with Node.js and Go frameworks, thanks to Starlette and async support.
- **Type-Driven Development:** Uses Python type hints for automatic request validation, serialization, and documentation.
- **Automatic OpenAPI/JSON Schema Generation:** Interactive Swagger UI and ReDoc documentation endpoints are generated automatically.
- **Dependency Injection:** Built-in system for managing request-scoped dependencies and shared components.
- **Security Utilities:** OAuth2/JWT patterns, HTTP authentication helpers, API key support.
- **WebSocket Support:** Full WebSocket support via Starlette.
- **Extensive Test Suite:** Comprehensive pytest-based tests with tooling for ruff, mypy, and coverage.

### Core Dependencies

| Dependency           | Minimum Version | Purpose                                  |
|----------------------|-----------------|------------------------------------------|
| `starlette`          | >=0.46.0        | ASGI web framework foundation            |
| `pydantic`           | >=2.7.0         | Data validation and serialization        |
| `typing-extensions`  | >=4.8.0         | Extended typing support                  |
| `typing-inspection`  | >=0.4.2         | Type inspection utilities                |
| `annotated-doc`      | >=0.0.2         | Documentation from type annotations      |

---

## Project Structure

```
fastapi-334443/
├── fastapi/                    # Core FastAPI library source
│   ├── __init__.py             # Package init, version, public exports
│   ├── applications.py         # FastAPI app class (main entrypoint)
│   ├── routing.py              # APIRouter and route handling
│   ├── params.py               # Parameter classes (Query, Path, Header, etc.)
│   ├── param_functions.py      # Parameter function helpers
│   ├── requests.py             # Request object extensions
│   ├── responses.py            # Response object extensions
│   ├── websockets.py           # WebSocket support
│   ├── cli.py                  # CLI entrypoint
│   ├── exceptions.py           # Exception classes
│   ├── exception_handlers.py   # Default exception handlers
│   ├── encoders.py             # JSON encoding utilities
│   ├── concurrency.py          # Async concurrency utilities
│   ├── datastructures.py       # Data structure helpers (UploadFile, etc.)
│   ├── logger.py               # Logging configuration
│   ├── types.py                # Type aliases
│   ├── utils.py                # General utilities
│   ├── sse.py                  # Server-Sent Events support
│   ├── staticfiles.py          # Static file serving
│   ├── templating.py           # Template rendering (Jinja2)
│   ├── testclient.py           # Test client for testing apps
│   ├── _compat/                # Compatibility layer modules
│   ├── dependencies/           # Dependency injection utilities
│   │   ├── models.py           # Dependency models
│   │   └── utils.py            # Dependency resolution and parameter extraction
│   ├── middleware/              # Middleware implementations
│   │   ├── cors.py             # CORS middleware
│   │   ├── gzip.py             # GZip middleware
│   │   ├── httpsredirect.py    # HTTPS redirect middleware
│   │   ├── trustedhost.py      # Trusted host middleware
│   │   ├── wsgi.py             # WSGI compatibility middleware
│   │   └── asyncexitstack.py   # Async exit stack middleware
│   ├── openapi/                # OpenAPI schema generation
│   │   ├── constants.py        # OpenAPI constants
│   │   ├── docs.py             # Swagger UI / ReDoc HTML generation
│   │   ├── models.py           # OpenAPI schema models
│   │   └── utils.py            # OpenAPI utility functions
│   └── security/               # Security utilities
│       ├── api_key.py          # API key authentication
│       ├── base.py             # Base security classes
│       ├── http.py             # HTTP authentication (Basic, Bearer, Digest)
│       ├── oauth2.py           # OAuth2 flows and password bearer
│       ├── open_id_connect_url.py  # OpenID Connect
│       └── utils.py            # Security utility functions
├── docs_src/                   # Documentation source examples (~457 items)
├── tests/                      # Test suite (~581 items)
├── scripts/                    # Developer and CI scripts
├── docs/                       # Generated/translated documentation
├── pyproject.toml              # Project configuration, dependencies, tools
├── README.md                   # Project README
├── LICENSE                     # MIT License
├── CONTRIBUTING.md             # Contribution guidelines
├── SECURITY.md                 # Security policy
└── CITATION.cff                # Citation metadata
```

---

## Getting Started

### Prerequisites

- **Python:** >= 3.10
- **Package Manager:** [uv](https://docs.astral.sh/uv/) (recommended) or pip

### Installation

```bash
# Install FastAPI with standard dependencies (includes uvicorn, httpx, jinja2, etc.)
pip install "fastapi[standard]"

# Or install minimal FastAPI
pip install fastapi
```

### Minimal Application

Create a file named `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    """Root endpoint returning a greeting."""
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    """Read an item by ID with an optional query parameter."""
    return {"item_id": item_id, "q": q}
```

### Running the Application

```bash
# Using the FastAPI CLI
fastapi dev main.py

# Or using Uvicorn directly
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Accessing Documentation

Once the application is running:

- **Swagger UI:** [http://localhost:8000/docs](http://localhost:8000/docs)
- **ReDoc:** [http://localhost:8000/redoc](http://localhost:8000/redoc)
- **OpenAPI JSON Schema:** [http://localhost:8000/openapi.json](http://localhost:8000/openapi.json)

---

## Core Concepts

### FastAPI Application

The `FastAPI` class (defined in `fastapi/applications.py`) is the main entrypoint. It extends Starlette's `Starlette` class with additional functionality for automatic OpenAPI schema generation, dependency injection, and interactive documentation.

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="A sample API built with FastAPI",
    version="1.0.0",
    docs_url="/docs",         # Swagger UI path (set to None to disable)
    redoc_url="/redoc",       # ReDoc path (set to None to disable)
    openapi_url="/openapi.json",  # OpenAPI schema path
)
```

**Key `FastAPI` constructor parameters:**

| Parameter         | Type       | Default          | Description                                     |
|-------------------|------------|------------------|-------------------------------------------------|
| `title`           | `str`      | `"FastAPI"`      | API title shown in documentation                |
| `description`     | `str`      | `""`             | API description (supports Markdown)             |
| `version`         | `str`      | `"0.1.0"`        | API version                                     |
| `docs_url`        | `str|None` | `"/docs"`        | Path for Swagger UI                             |
| `redoc_url`       | `str|None` | `"/redoc"`       | Path for ReDoc documentation                    |
| `openapi_url`     | `str|None` | `"/openapi.json"`| Path for OpenAPI JSON schema                    |
| `openapi_tags`    | `list`     | `None`           | Tag metadata for organizing endpoints           |
| `lifespan`        | `Lifespan` | `None`           | Async context manager for startup/shutdown       |

### Routing and Path Operations

Routes are defined using decorators on the `FastAPI` app instance or on an `APIRouter`.

#### Using the App Directly

```python
@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """Retrieve a user by their ID."""
    return {"user_id": user_id}

@app.post("/users/", status_code=201)
async def create_user(name: str):
    """Create a new user."""
    return {"name": name}
```

#### Using APIRouter for Modular Applications

```python
from fastapi import APIRouter

router = APIRouter(prefix="/items", tags=["items"])

@router.get("/")
async def list_items():
    """List all items."""
    return [{"item": "Foo"}, {"item": "Bar"}]

@router.get("/{item_id}")
async def get_item(item_id: int):
    """Get a specific item by ID."""
    return {"item_id": item_id}

# In your main app:
app.include_router(router)
```

**Supported HTTP Methods:**

- `@app.get()` / `@router.get()`
- `@app.post()` / `@router.post()`
- `@app.put()` / `@router.put()`
- `@app.delete()` / `@router.delete()`
- `@app.patch()` / `@router.patch()`
- `@app.options()` / `@router.options()`
- `@app.head()` / `@router.head()`
- `@app.trace()` / `@router.trace()`

### Request Parameters

FastAPI uses Python type hints and special parameter functions to extract and validate request data. The parameter classes are defined in `fastapi/params.py` and the helper functions in `fastapi/param_functions.py`.

#### Path Parameters

```python
from fastapi import Path

@app.get("/items/{item_id}")
async def read_item(
    item_id: int = Path(..., title="Item ID", ge=1, description="The ID of the item")
):
    return {"item_id": item_id}
```

#### Query Parameters

```python
from fastapi import Query

@app.get("/items/")
async def list_items(
    skip: int = Query(0, ge=0, description="Number of items to skip"),
    limit: int = Query(10, le=100, description="Max number of items to return"),
):
    return {"skip": skip, "limit": limit}
```

#### Header Parameters

```python
from fastapi import Header

@app.get("/items/")
async def read_items(user_agent: str | None = Header(None)):
    return {"User-Agent": user_agent}
```

#### Cookie Parameters

```python
from fastapi import Cookie

@app.get("/items/")
async def read_items(session_id: str | None = Cookie(None)):
    return {"session_id": session_id}
```

#### Request Body (using Pydantic models)

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str = Field(..., description="The name of the item")
    price: float = Field(..., gt=0, description="The price of the item")
    description: str | None = Field(None, description="Optional description")

@app.post("/items/")
async def create_item(item: Item):
    return item
```

#### File Uploads

```python
from fastapi import File, UploadFile

@app.post("/uploadfile/")
async def upload_file(file: UploadFile = File(..., description="The file to upload")):
    return {"filename": file.filename}
```

#### Form Data

```python
from fastapi import Form

@app.post("/login/")
async def login(username: str = Form(...), password: str = Form(...)):
    return {"username": username}
```

### Dependency Injection

FastAPI's dependency injection system (implemented in `fastapi/dependencies/`) allows you to define reusable components that are resolved per-request.

```python
from fastapi import Depends

async def get_db():
    """Provide a database session for the request lifecycle."""
    db = DatabaseSession()
    try:
        yield db
    finally:
        db.close()

async def get_current_user(db=Depends(get_db)):
    """Retrieve the current authenticated user."""
    # Authentication logic here
    return {"user": "authenticated_user"}

@app.get("/users/me")
async def read_current_user(user=Depends(get_current_user)):
    return user
```

**Key features:**

- Dependencies can be **functions** or **callable classes**.
- Dependencies can depend on **other dependencies** (nested resolution).
- **Generator dependencies** (using `yield`) support cleanup logic.
- Dependencies are **cached per-request** by default (same dependency used multiple times returns the same instance).

### Security

FastAPI provides security utilities in `fastapi/security/` that integrate with the OpenAPI specification for automatic documentation of authentication methods.

#### OAuth2 with Password Flow

```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """Authenticate and return an access token."""
    # Validate credentials and create token
    return {"access_token": "token_value", "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(token: str = Depends(oauth2_scheme)):
    """Get current user using OAuth2 bearer token."""
    return {"token": token}
```

#### API Key Authentication

```python
from fastapi.security import APIKeyHeader

api_key_header = APIKeyHeader(name="X-API-Key")

@app.get("/protected")
async def protected_route(api_key: str = Depends(api_key_header)):
    return {"api_key": api_key}
```

#### HTTP Basic Authentication

```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials

security = HTTPBasic()

@app.get("/secure")
async def secure_endpoint(credentials: HTTPBasicCredentials = Depends(security)):
    return {"username": credentials.username}
```

**Available Security Classes:**

| Class                      | Module                      | Description                            |
|----------------------------|-----------------------------|----------------------------------------|
| `OAuth2PasswordBearer`     | `fastapi.security.oauth2`   | OAuth2 password bearer flow            |
| `OAuth2AuthorizationCodeBearer` | `fastapi.security.oauth2` | OAuth2 authorization code flow        |
| `APIKeyHeader`             | `fastapi.security.api_key`  | API key via header                     |
| `APIKeyQuery`              | `fastapi.security.api_key`  | API key via query parameter            |
| `APIKeyCookie`             | `fastapi.security.api_key`  | API key via cookie                     |
| `HTTPBasic`                | `fastapi.security.http`     | HTTP Basic authentication              |
| `HTTPBearer`               | `fastapi.security.http`     | HTTP Bearer token authentication       |
| `HTTPDigest`               | `fastapi.security.http`     | HTTP Digest authentication             |
| `OpenIdConnect`            | `fastapi.security.open_id_connect_url` | OpenID Connect            |

### Middleware

Middleware is added to the FastAPI application to process requests and responses globally. FastAPI re-exports Starlette middleware and provides additional options.

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.add_middleware(GZipMiddleware, minimum_size=1000)
```

**Available Middleware:**

| Middleware              | Module                            | Description                          |
|-------------------------|-----------------------------------|--------------------------------------|
| `CORSMiddleware`        | `fastapi.middleware.cors`         | Cross-Origin Resource Sharing        |
| `GZipMiddleware`        | `fastapi.middleware.gzip`         | GZip response compression            |
| `HTTPSRedirectMiddleware` | `fastapi.middleware.httpsredirect` | Redirect HTTP to HTTPS            |
| `TrustedHostMiddleware` | `fastapi.middleware.trustedhost`  | Restrict allowed host headers        |
| `WSGIMiddleware`        | `fastapi.middleware.wsgi`         | Mount WSGI applications              |

#### Custom Middleware

```python
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi import Request

class TimingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        import time
        start = time.time()
        response = await call_next(request)
        duration = time.time() - start
        response.headers["X-Process-Time"] = str(duration)
        return response

app.add_middleware(TimingMiddleware)
```

### OpenAPI and Documentation

FastAPI automatically generates an OpenAPI schema from your route definitions, type hints, and Pydantic models. The schema generation logic lives in `fastapi/openapi/`.

#### Customizing OpenAPI Schema

```python
app = FastAPI(
    title="My API",
    description="Full API description with **Markdown** support.",
    version="2.0.0",
    openapi_tags=[
        {"name": "users", "description": "Operations with users"},
        {"name": "items", "description": "Manage items"},
    ],
    contact={"name": "API Support", "email": "support@example.com"},
    license_info={"name": "MIT", "url": "https://opensource.org/licenses/MIT"},
)
```

#### Conditional OpenAPI (disable in production)

```python
import os
from fastapi import FastAPI

app = FastAPI(
    openapi_url="/openapi.json" if os.getenv("ENVIRONMENT") != "production" else None
)
```

---

## CLI Usage

FastAPI provides a CLI tool (defined in `fastapi/cli.py`) for common development tasks.

```bash
# Start development server with auto-reload
fastapi dev main.py

# Run in production mode
fastapi run main.py

# Show help
fastapi --help
```

The CLI is provided by the `fastapi-cli` package and is registered as the `fastapi` console script entry point in `pyproject.toml`.

---

## Configuration and Environment

### Environment Variables

The project uses a `.env` file for configuration. Key variables include:

| Variable             | Description                                  | Default       |
|----------------------|----------------------------------------------|---------------|
| `UVICORN_HOST`       | Host to bind the server to                   | `0.0.0.0`     |
| `PORT`               | Port number for the server                   | `3001`        |
| `UVICORN_WORKERS`    | Number of Uvicorn worker processes           | `1`           |
| `ALLOWED_ORIGINS`    | Comma-separated list of allowed CORS origins | `*`           |
| `ALLOWED_HEADERS`    | Comma-separated list of allowed headers      | `*`           |
| `ALLOWED_METHODS`    | Comma-separated list of allowed HTTP methods | `*`           |
| `CORS_MAX_AGE`       | CORS preflight cache duration (seconds)      | `3600`        |
| `NODE_ENV`           | Environment mode (development/production)    | `development` |
| `REQUEST_TIMEOUT_MS` | Request timeout in milliseconds              | `30000`       |
| `RATE_LIMIT_WINDOW_S`| Rate limiting window in seconds              | `60`          |
| `RATE_LIMIT_MAX`     | Maximum requests per rate limit window       | `100`         |

### Using Pydantic Settings

For type-safe configuration management, use `pydantic-settings`:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My FastAPI App"
    debug: bool = False
    database_url: str = "sqlite:///./app.db"

    class Config:
        env_file = ".env"

settings = Settings()
```

---

## Testing

### Test Infrastructure

FastAPI uses **pytest** as its testing framework. The test suite is located in the `tests/` directory with over 580 test files covering tutorials, request parameters, and core functionality.

### Writing Tests

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_root():
    """Test the root endpoint returns a greeting."""
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

def test_read_item():
    """Test reading an item by ID."""
    response = client.get("/items/42?q=test")
    assert response.status_code == 200
    data = response.json()
    assert data["item_id"] == 42
    assert data["q"] == "test"
```

### Async Testing

```python
import pytest
from httpx import ASGITransport, AsyncClient
from main import app

@pytest.mark.anyio
async def test_root_async():
    """Test root endpoint using async client."""
    async with AsyncClient(
        transport=ASGITransport(app=app), base_url="http://test"
    ) as client:
        response = await client.get("/")
        assert response.status_code == 200
```

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=fastapi --cov-report=html

# Run a specific test file
pytest tests/test_tutorial/test_first_steps/test_tutorial001.py

# Run tests in parallel
pytest -n auto
```

### Testing Dependencies Override

```python
from fastapi.testclient import TestClient

def override_get_db():
    """Override database dependency for testing."""
    return MockDatabase()

app.dependency_overrides[get_db] = override_get_db
client = TestClient(app)
```

---

## Development Workflow

### Setting Up the Development Environment

```bash
# Clone the repository
git clone https://github.com/fastapi/fastapi.git
cd fastapi

# Install development dependencies (using uv)
uv sync --group dev

# Or using pip
pip install -e ".[all]"
```

### Code Quality Tools

The project uses several tools configured in `pyproject.toml`:

| Tool       | Purpose                        | Configuration Key     |
|------------|--------------------------------|-----------------------|
| **ruff**   | Linting and formatting         | `[tool.ruff]`        |
| **mypy**   | Static type checking           | `[tool.mypy]`        |
| **pytest** | Test execution                 | `[tool.pytest]`      |
| **coverage** | Code coverage reporting      | `[tool.coverage]`    |

```bash
# Run linting
ruff check .

# Run type checking
mypy fastapi

# Run formatter
ruff format .
```

### Build System

The project uses **PDM** as its build backend:

```bash
# Build the package
pdm build

# The version is sourced from fastapi/__init__.py
```

---

## API Reference Summary

### Public Exports from `fastapi`

The following symbols are exported from the `fastapi` package (via `fastapi/__init__.py`):

| Symbol                   | Type       | Description                                  |
|--------------------------|------------|----------------------------------------------|
| `FastAPI`                | Class      | Main application class                       |
| `APIRouter`              | Class      | Router for modular route organization        |
| `Request`                | Class      | HTTP request object                          |
| `Response`               | Class      | HTTP response object                         |
| `WebSocket`              | Class      | WebSocket connection object                  |
| `WebSocketDisconnect`    | Exception  | WebSocket disconnection exception            |
| `HTTPException`          | Exception  | HTTP error exception                         |
| `UploadFile`             | Class      | File upload data structure                   |
| `BackgroundTasks`        | Class      | Background task manager                      |
| `Depends`                | Function   | Dependency injection marker                  |
| `Query`                  | Function   | Query parameter declaration                  |
| `Path`                   | Function   | Path parameter declaration                   |
| `Body`                   | Function   | Request body declaration                     |
| `Cookie`                 | Function   | Cookie parameter declaration                 |
| `Header`                 | Function   | Header parameter declaration                 |
| `Form`                   | Function   | Form data declaration                        |
| `File`                   | Function   | File upload declaration                      |
| `Security`               | Function   | Security dependency marker                   |
| `status`                 | Module     | HTTP status code constants (from Starlette)  |

---

## Troubleshooting

### Common Issues

#### 1. Import Errors

```
ModuleNotFoundError: No module named 'fastapi'
```

**Solution:** Ensure FastAPI is installed in your active Python environment:
```bash
pip install fastapi
```

#### 2. Uvicorn Not Found

```
Command 'uvicorn' not found
```

**Solution:** Install with standard extras:
```bash
pip install "fastapi[standard]"
```

#### 3. Pydantic Validation Errors

If you see `ValidationError` responses, check that your request data matches the Pydantic model schema. Use the interactive Swagger UI at `/docs` to test requests with the correct format.

#### 4. CORS Errors in Browser

**Solution:** Add CORS middleware to your application:
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

#### 5. Async vs Sync Endpoints

- Use `async def` for endpoints that perform async I/O operations.
- Use regular `def` for CPU-bound or synchronous operations (FastAPI will run them in a thread pool automatically).

```python
# Async endpoint (for async I/O)
@app.get("/async-items/")
async def read_items():
    items = await async_db_query()
    return items

# Sync endpoint (auto-threaded by FastAPI)
@app.get("/sync-items/")
def read_items_sync():
    items = sync_db_query()
    return items
```

---

## Additional Resources

- **Official Documentation:** [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- **GitHub Repository:** [https://github.com/fastapi/fastapi](https://github.com/fastapi/fastapi)
- **PyPI Package:** [https://pypi.org/project/fastapi/](https://pypi.org/project/fastapi/)
- **Starlette Documentation:** [https://www.starlette.io/](https://www.starlette.io/)
- **Pydantic Documentation:** [https://docs.pydantic.dev/](https://docs.pydantic.dev/)
- **Contributing Guide:** See `CONTRIBUTING.md` in the project root.
- **Security Policy:** See `SECURITY.md` in the project root.

---

*This document was generated for the FastAPI v0.135.1 codebase. For the most up-to-date information, refer to the official documentation.*
