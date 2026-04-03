---
name: api-integration
description: Auto-activating API integration skill — HTTP clients, OAuth flows, retry logic, circuit breakers, webhooks, polling, SDK wrappers, and connection pooling
origin: jeremylongshore/claude-code-plugins-plus-skills/16-api-integration
tools: Read, Write, Edit, Bash, Grep
---

# API Integration Skill

Automated assistance for integrating with third-party APIs: HTTP client configuration, authentication, reliability patterns (retries, circuit breakers), webhooks, and SDK wrappers.

## When to Activate

Activates automatically when you mention: API client, HTTP client, OAuth, webhook, retry, circuit breaker, polling, rate limit, SDK wrapper, connection pool, request interceptor, response transformer, async API, long polling, server-sent events, WebSocket client.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `api-client-generator` | Typed API client class from OpenAPI spec |
| `oauth-client-setup` | OAuth 2.0 client with PKCE and token refresh |
| `oauth-callback-handler` | Authorization callback and token exchange |
| `retry-logic-helper` | Exponential backoff with jitter |
| `circuit-breaker-setup` | Circuit breaker pattern implementation |
| `request-interceptor-creator` | Auth header injection, request logging |
| `response-transformer` | Response normalization middleware |
| `webhook-receiver-generator` | Webhook endpoint with signature validation |
| `webhook-sender-creator` | Reliable webhook delivery with retries |
| `webhook-signature-validator` | HMAC signature verification |
| `webhook-retry-handler` | Dead letter queue for failed deliveries |
| `polling-mechanism-setup` | Efficient polling with exponential backoff |
| `long-polling-handler` | Long-polling server and client implementation |
| `server-sent-events-setup` | SSE endpoint and client |
| `websocket-client-creator` | Reconnecting WebSocket client |
| `batch-request-handler` | Batching individual requests into bulk calls |
| `connection-pooling-config` | HTTP connection pool tuning |
| `timeout-handler` | Request timeout with graceful degradation |
| `error-mapping-helper` | Map API error codes to domain errors |
| `api-health-checker` | Dependency health check aggregator |
| `sdk-wrapper-creator` | Type-safe wrapper around a REST API |
| `api-response-cacher` | Response caching with TTL and invalidation |
| `integration-test-generator` | Contract tests for third-party API integrations |

## Retry with Exponential Backoff

```python
import asyncio
import random
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1.0, max_delay=60.0, exceptions=(Exception,)):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_retries + 1):
                try:
                    return await func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_retries:
                        raise
                    delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
                    await asyncio.sleep(delay)
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3, base_delay=1.0, exceptions=(httpx.HTTPStatusError, httpx.ConnectError))
async def call_api(client: httpx.AsyncClient, url: str) -> dict:
    response = await client.get(url, timeout=30.0)
    response.raise_for_status()
    return response.json()
```

## Circuit Breaker

```python
from enum import Enum
from datetime import datetime, timedelta

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60, success_threshold=2):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.success_threshold = success_threshold
        self.failures = 0
        self.successes = 0
        self.state = CircuitState.CLOSED
        self.opened_at = None

    async def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if datetime.now() - self.opened_at > timedelta(seconds=self.recovery_timeout):
                self.state = CircuitState.HALF_OPEN
                self.successes = 0
            else:
                raise Exception("Circuit breaker is OPEN")
        try:
            result = await func(*args, **kwargs)
            self._on_success()
            return result
        except Exception:
            self._on_failure()
            raise

    def _on_success(self):
        self.failures = 0
        if self.state == CircuitState.HALF_OPEN:
            self.successes += 1
            if self.successes >= self.success_threshold:
                self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failures += 1
        if self.failures >= self.failure_threshold:
            self.state = CircuitState.OPEN
            self.opened_at = datetime.now()
```

## Webhook Signature Validation

```python
import hmac
import hashlib

def validate_webhook_signature(payload: bytes, signature_header: str, secret: str) -> bool:
    """Validate HMAC-SHA256 webhook signature (GitHub/Stripe style)."""
    expected = hmac.new(secret.encode(), payload, hashlib.sha256).hexdigest()
    received = signature_header.removeprefix("sha256=")
    return hmac.compare_digest(expected, received)  # constant-time comparison
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `16-api-integration` (25 skills)
