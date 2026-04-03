---
name: backend-dev
description: Auto-activating backend development skill — Express/Fastify/NestJS/Django/FastAPI/Gin, ORM models, Kafka, Redis, GraphQL, gRPC, WebSockets, background workers, and middleware
origin: jeremylongshore/claude-code-plugins-plus-skills/06-backend-dev
tools: Read, Write, Edit, Bash, Grep
---

# Backend Dev Skill

Automated assistance for backend development across frameworks and languages: API servers, database modeling, message queues, caching, and communication protocols.

## When to Activate

Activates automatically when you mention: Express, Fastify, NestJS, Django, FastAPI, Gin, Prisma, Sequelize, TypeORM, Kafka, Redis, RabbitMQ, GraphQL, gRPC, WebSocket, background worker, cron job, middleware, rate limiting, request validation.

## Covered Skills (30 total)

| Framework | Skill |
|-----------|-------|
| Node.js | `express-server-creator`, `fastify-server-creator`, `nestjs-module-creator` |
| Python | `django-app-generator`, `flask-api-creator`, `fastapi-endpoint-builder` |
| Go | `gin-handler-creator` |
| ORM/DB | `prisma-entity-generator`, `sequelize-model-creator`, `typeorm-entity-creator`, `sql-migration-generator`, `database-schema-designer` |
| Queues | `kafka-producer-creator`, `kafka-consumer-creator`, `rabbitmq-queue-setup` |
| Cache | `redis-cache-manager`, `memcached-config-helper` |
| API | `graphql-resolver-creator`, `grpc-service-generator`, `websocket-handler-setup` |
| Workers | `background-worker-creator`, `cron-job-scheduler` |
| Middleware | `error-handler-middleware`, `rate-limit-middleware`, `request-validator-generator` |

## FastAPI Endpoint Pattern

```python
from fastapi import FastAPI, Depends, HTTPException, status
from pydantic import BaseModel
from typing import Annotated

app = FastAPI()

class ItemCreate(BaseModel):
    name: str
    price: float
    description: str | None = None

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float
    model_config = {"from_attributes": True}

@app.post("/items", response_model=ItemResponse, status_code=status.HTTP_201_CREATED)
async def create_item(
    item: ItemCreate,
    db: Annotated[Session, Depends(get_db)],
    current_user: Annotated[User, Depends(get_current_user)]
) -> ItemResponse:
    db_item = Item(**item.model_dump(), owner_id=current_user.id)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item
```

## Express Middleware Stack

```typescript
import express from 'express'
import helmet from 'helmet'
import rateLimit from 'express-rate-limit'
import { z } from 'zod'

const app = express()
app.use(helmet())
app.use(express.json({ limit: '1mb' }))
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }))

// Request validation middleware
const validate = (schema: z.ZodSchema) => (req: Request, res: Response, next: NextFunction) => {
  const result = schema.safeParse(req.body)
  if (!result.success) return res.status(400).json({ errors: result.error.flatten() })
  req.body = result.data
  next()
}

// Error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack)
  res.status(500).json({ error: 'Internal server error' })
})
```

## Prisma Schema Pattern

```prisma
model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String?
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  posts     Post[]
  @@index([email])
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  @@index([authorId, published])
}
```

## Redis Cache Pattern

```python
import redis
import json
from functools import wraps

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def cache(ttl: int = 300, key_prefix: str = ""):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            cache_key = f"{key_prefix}:{func.__name__}:{hash(str(args) + str(kwargs))}"
            cached = r.get(cache_key)
            if cached:
                return json.loads(cached)
            result = await func(*args, **kwargs)
            r.setex(cache_key, ttl, json.dumps(result))
            return result
        return wrapper
    return decorator
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `06-backend-dev` (30 skills)
