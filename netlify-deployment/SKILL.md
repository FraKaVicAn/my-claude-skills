---
name: netlify-deployment
description: Netlify deployment, serverless functions, edge functions, Blobs storage, managed Postgres, image CDN, forms, AI gateway. Auto-activates for Netlify projects.
origin: VoltAgent/awesome-agent-skills (netlify)
tools: Read, Write, Edit, Bash
---

# Netlify Deployment Skills

Official Netlify skills covering functions, edge, storage, and deployment.

## CLI & Deployment

```bash
# Install
npm install -g netlify-cli

# Login & link
netlify login
netlify link

# Local dev (with functions)
netlify dev

# Deploy preview
netlify deploy

# Deploy to production
netlify deploy --prod

# Environment variables
netlify env:set DATABASE_URL "postgres://..."
netlify env:list
netlify env:import .env.production
```

## netlify.toml

```toml
[build]
  command = "npm run build"
  publish = ".next"        # or "dist", "out", "public"

[build.environment]
  NODE_VERSION = "20"

[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

[functions]
  directory = "netlify/functions"
  node_bundler = "esbuild"

[[edge_functions]]
  path = "/api/*"
  function = "auth-middleware"
```

## Serverless Functions

```ts
// netlify/functions/hello.ts
import type { Handler, HandlerEvent, HandlerContext } from '@netlify/functions';

export const handler: Handler = async (event: HandlerEvent, context: HandlerContext) => {
  const { httpMethod, body, queryStringParameters } = event;

  if (httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  const data = JSON.parse(body ?? '{}');

  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message: 'Hello!', data }),
  };
};
```

### Background Functions (async, up to 15 min)

```ts
// netlify/functions/process-report.ts
export const handler: Handler = async (event) => {
  // Return immediately, process in background
  if (event.headers['x-netlify-background'] !== 'true') {
    return {
      statusCode: 202,
      body: JSON.stringify({ message: 'Processing started' }),
    };
  }

  // Long-running work here
  await generateReport();
  return { statusCode: 200, body: 'Done' };
};
```

## Edge Functions

```ts
// netlify/edge-functions/auth-middleware.ts
import type { Context } from '@netlify/edge-functions';

export default async (request: Request, context: Context) => {
  const token = request.headers.get('Authorization')?.split(' ')[1];

  if (!token) {
    return new Response('Unauthorized', { status: 401 });
  }

  // Geolocation available at the edge
  const country = context.geo?.country?.code ?? 'US';

  // Modify request/response headers
  const response = await context.next();
  response.headers.set('X-Country', country);
  return response;
};
```

## Netlify Blobs (Key-Value Storage)

```ts
import { getStore } from '@netlify/blobs';

// In a function
const store = getStore('my-store');

// Write
await store.set('key', 'value');
await store.setJSON('user:123', { name: 'Alice', role: 'admin' });

// Read
const value = await store.get('key');
const user = await store.get('user:123', { type: 'json' });

// List
const { blobs } = await store.list({ prefix: 'user:' });

// Delete
await store.delete('key');
```

## Netlify DB (Managed Postgres)

```bash
# Create DB (auto-branches per deploy preview)
netlify db:create

# Run migrations
netlify db:migrate
```

```ts
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

export const handler: Handler = async () => {
  const users = await sql`SELECT * FROM users LIMIT 10`;
  return { statusCode: 200, body: JSON.stringify(users) };
};
```

## Image CDN

```html
<!-- Automatic optimization via Netlify Image CDN -->
<img src="/.netlify/images?url=/my-image.jpg&w=800&h=600&fit=cover&fm=webp" />
```

```ts
// In code
const optimizedUrl = `/.netlify/images?url=${encodeURIComponent(originalUrl)}&w=800&fm=webp&q=80`;
```

## Forms

```html
<!-- HTML form — Netlify auto-handles submissions -->
<form name="contact" method="POST" data-netlify="true" netlify-honeypot="bot-field">
  <input type="hidden" name="form-name" value="contact" />
  <input type="hidden" name="bot-field" />
  <input type="text" name="name" required />
  <input type="email" name="email" required />
  <textarea name="message"></textarea>
  <button type="submit">Send</button>
</form>
```

## AI Gateway

```ts
// Use Netlify AI Gateway to access multiple AI providers
const response = await fetch('/.netlify/functions/ai-gateway', {
  method: 'POST',
  body: JSON.stringify({
    model: 'claude-sonnet-4-6',
    messages: [{ role: 'user', content: 'Hello!' }],
  }),
});
```

## Caching

```ts
// Fine-grained cache control from functions
export const handler: Handler = async (event) => {
  const data = await fetchExpensiveData();

  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'public, max-age=0, stale-while-revalidate=86400',
      'Netlify-CDN-Cache-Control': 'public, max-age=86400, stale-while-revalidate=86400',
      'Netlify-Cache-Tag': 'products, featured',
    },
    body: JSON.stringify(data),
  };
};
```

```bash
# Purge cache by tag
netlify cache:purge --tag products
```

## Sources
- netlify/netlify-functions
- netlify/netlify-edge-functions
- netlify/netlify-blobs
- netlify/netlify-db
- netlify/netlify-image-cdn
- netlify/netlify-forms
- netlify/netlify-caching
- netlify/netlify-config
- netlify/netlify-cli-and-deploy
- netlify/netlify-ai-gateway
