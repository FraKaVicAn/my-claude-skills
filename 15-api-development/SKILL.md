---
name: api-development
description: Auto-activating API development skill — REST design, OpenAPI/Swagger, GraphQL schemas, gRPC, authentication, rate limiting, pagination, caching, versioning, and API mocking
origin: jeremylongshore/claude-code-plugins-plus-skills/15-api-development
tools: Read, Write, Edit, Bash, Grep
---

# API Development Skill

Automated assistance for designing and building APIs: REST conventions, OpenAPI specs, GraphQL schemas, authentication patterns, performance, and documentation.

## When to Activate

Activates automatically when you mention: REST API, endpoint, OpenAPI, Swagger, GraphQL, gRPC, authentication, rate limiting, pagination, caching, ETag, versioning, HATEOAS, API mock, API design, status code, request body, response schema.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `rest-endpoint-designer` | RESTful resource naming and HTTP method mapping |
| `openapi-spec-generator` | OpenAPI 3.1 YAML spec from route descriptions |
| `swagger-doc-creator` | Swagger UI integration and annotations |
| `graphql-schema-generator` | GraphQL SDL type definitions |
| `graphql-resolver-creator` | Resolver implementation with DataLoader |
| `graphql-mutation-builder` | Mutations with input types and error handling |
| `graphql-subscription-setup` | Real-time subscriptions with WebSockets |
| `api-key-auth-setup` | API key generation, validation, rotation |
| `bearer-token-validator` | JWT bearer token middleware |
| `api-caching-strategy` | Cache-Control, ETag, conditional requests |
| `api-rate-limiting-config` | Token bucket, sliding window configs |
| `pagination-helper` | Cursor and offset pagination patterns |
| `filtering-query-builder` | Filter, sort, search query parameter patterns |
| `versioning-strategy-helper` | URL, header, and media type versioning |
| `api-mock-generator` | Mock server with Prism or MSW |
| `hypermedia-link-generator` | HATEOAS link generation |
| `status-code-recommender` | Correct HTTP status codes for each scenario |
| `request-body-validator` | JSON Schema request validation middleware |
| `response-schema-generator` | Consistent response envelope patterns |

## REST Resource Design

```
# Resource naming conventions
GET    /users              → list users (paginated)
POST   /users              → create user
GET    /users/{id}         → get user
PUT    /users/{id}         → replace user
PATCH  /users/{id}         → partial update
DELETE /users/{id}         → delete user

GET    /users/{id}/orders  → list user's orders
POST   /users/{id}/orders  → create order for user

# Filter, sort, paginate
GET /products?category=electronics&price[gte]=100&price[lte]=500
              &sort=-rating,price
              &page[cursor]=eyJpZCI6MTB9&page[size]=20
              &fields=id,name,price,rating
```

## OpenAPI 3.1 Snippet

```yaml
openapi: 3.1.0
info:
  title: My API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      tags: [Users]
      security: [{bearerAuth: []}]
      parameters:
        - name: id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema: {$ref: '#/components/schemas/User'}
        '404': {$ref: '#/components/responses/NotFound'}
components:
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id: {type: string, format: uuid}
        email: {type: string, format: email}
        name: {type: string}
        createdAt: {type: string, format: date-time}
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## GraphQL Schema

```graphql
type Query {
  user(id: ID!): User
  users(filter: UserFilter, pagination: Pagination): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

type User {
  id: ID!
  email: String!
  name: String
  orders(first: Int, after: String): OrderConnection!
  createdAt: DateTime!
}

input CreateUserInput {
  email: String!
  name: String
  password: String!
}

type CreateUserPayload {
  user: User
  errors: [UserError!]!
}
```

## Cursor Pagination Pattern

```python
import base64
import json

def encode_cursor(data: dict) -> str:
    return base64.b64encode(json.dumps(data).encode()).decode()

def decode_cursor(cursor: str) -> dict:
    return json.loads(base64.b64decode(cursor.encode()).decode())

def paginate(query, after_cursor: str | None, first: int = 20):
    if after_cursor:
        cursor_data = decode_cursor(after_cursor)
        query = query.where("id > :last_id", last_id=cursor_data["id"])
    items = query.limit(first + 1).all()
    has_next = len(items) > first
    edges = [{"node": item, "cursor": encode_cursor({"id": item.id})} for item in items[:first]]
    return {"edges": edges, "pageInfo": {"hasNextPage": has_next, "endCursor": edges[-1]["cursor"] if edges else None}}
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `15-api-development` (26 skills)
