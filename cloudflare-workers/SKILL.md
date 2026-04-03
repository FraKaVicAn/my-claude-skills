---
name: cloudflare-workers
description: Cloudflare Workers, Durable Objects, AI agents, MCP servers, KV, R2, D1, Queues, and Wrangler deployment. Auto-activates for Cloudflare, Workers, or Wrangler projects.
origin: VoltAgent/awesome-agent-skills (cloudflare)
tools: Read, Write, Edit, Bash
---

# Cloudflare Workers & Agents Skills

Official skills from the Cloudflare team covering Workers, Durable Objects, AI agents, and MCP servers.

## Workers Basics

```ts
// src/index.ts
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === '/api/hello') {
      return Response.json({ message: 'Hello from Cloudflare Workers!' });
    }

    return new Response('Not Found', { status: 404 });
  },
};

interface Env {
  MY_KV: KVNamespace;
  MY_BUCKET: R2Bucket;
  MY_DB: D1Database;
  MY_QUEUE: Queue;
}
```

## KV Storage

```ts
// Write
await env.MY_KV.put('key', 'value', { expirationTtl: 3600 });
await env.MY_KV.put('json-key', JSON.stringify({ data: 'value' }));

// Read
const value = await env.MY_KV.get('key');
const json = await env.MY_KV.get('json-key', 'json');

// List
const list = await env.MY_KV.list({ prefix: 'user:', limit: 100 });

// Delete
await env.MY_KV.delete('key');
```

## R2 Object Storage

```ts
// Upload
await env.MY_BUCKET.put('file.txt', 'content', {
  httpMetadata: { contentType: 'text/plain' },
  customMetadata: { userId: '123' },
});

// Download
const object = await env.MY_BUCKET.get('file.txt');
if (!object) return new Response('Not found', { status: 404 });
return new Response(object.body, { headers: { 'Content-Type': object.httpMetadata?.contentType ?? 'application/octet-stream' } });

// Delete
await env.MY_BUCKET.delete('file.txt');
```

## D1 Database (SQLite)

```ts
// Query
const result = await env.MY_DB.prepare(
  'SELECT * FROM users WHERE id = ?'
).bind(userId).first();

// Batch
const [users, posts] = await env.MY_DB.batch([
  env.MY_DB.prepare('SELECT * FROM users LIMIT 10'),
  env.MY_DB.prepare('SELECT * FROM posts WHERE user_id = ?').bind(userId),
]);

// Execute (migrations)
await env.MY_DB.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id TEXT PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    created_at INTEGER DEFAULT (unixepoch())
  )
`);
```

## Durable Objects (Stateful)

```ts
// Durable Object class
export class ChatRoom {
  private state: DurableObjectState;
  private sessions: WebSocket[] = [];

  constructor(state: DurableObjectState, env: Env) {
    this.state = state;
  }

  async fetch(request: Request): Promise<Response> {
    if (request.headers.get('Upgrade') === 'websocket') {
      const [client, server] = Object.values(new WebSocketPair());
      server.accept();
      this.sessions.push(server);

      server.addEventListener('message', (msg) => {
        // Broadcast to all sessions
        this.sessions.forEach(s => s.send(msg.data));
      });

      return new Response(null, { status: 101, webSocket: client });
    }

    // SQLite storage within Durable Object
    const count = await this.state.storage.get<number>('message_count') ?? 0;
    return Response.json({ count });
  }
}

// Use in Worker
const id = env.CHAT_ROOM.idFromName('room-1');
const stub = env.CHAT_ROOM.get(id);
return stub.fetch(request);
```

## AI Agents with Cloudflare Agents SDK

```ts
import { Agent, AgentContext } from 'agents';

export class MyAgent extends Agent<Env> {
  async onRequest(request: Request): Promise<Response> {
    // Access persistent state
    const history = await this.state.get<Message[]>('history') ?? [];

    // Call AI model
    const response = await this.env.AI.run('@cf/meta/llama-3.1-8b-instruct', {
      messages: [...history, { role: 'user', content: await request.text() }],
    });

    // Save state
    await this.state.set('history', [...history, { role: 'assistant', content: response.response }]);

    return Response.json(response);
  }

  // Schedule tasks
  async onSchedule(scheduled: ScheduledEvent): Promise<void> {
    await this.schedule('daily-report', '0 9 * * *', async () => {
      // Run daily report
    });
  }
}
```

## MCP Server on Cloudflare

```ts
import { McpAgent } from 'agents/mcp';
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { z } from 'zod';

export class MyMCP extends McpAgent<Env> {
  server = new McpServer({ name: 'My MCP Server', version: '1.0.0' });

  async init() {
    // Register tools
    this.server.tool(
      'get-weather',
      { city: z.string() },
      async ({ city }) => {
        const data = await fetch(`https://api.weather.com/${city}`).then(r => r.json());
        return { content: [{ type: 'text', text: JSON.stringify(data) }] };
      }
    );

    // Register resources
    this.server.resource('config', 'config://app', async () => ({
      contents: [{ uri: 'config://app', text: JSON.stringify(this.env.CONFIG) }],
    }));
  }
}

export default {
  fetch: MyMCP.serve('/mcp'),
};
```

## Wrangler CLI

```bash
# Init
npm create cloudflare@latest my-worker

# Dev server
wrangler dev

# Deploy
wrangler deploy

# Tail logs
wrangler tail

# KV operations
wrangler kv key put --binding=MY_KV "key" "value"
wrangler kv key get --binding=MY_KV "key"

# D1 migrations
wrangler d1 execute MY_DB --file=./schema.sql
wrangler d1 execute MY_DB --command="SELECT * FROM users"

# R2 operations
wrangler r2 object put MY_BUCKET/file.txt --file=./file.txt

# Queues
wrangler queues create my-queue
```

### wrangler.toml
```toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-09-23"

[[kv_namespaces]]
binding = "MY_KV"
id = "abc123"

[[d1_databases]]
binding = "MY_DB"
database_name = "my-db"
database_id = "abc123"

[[r2_buckets]]
binding = "MY_BUCKET"
bucket_name = "my-bucket"

[[queues.producers]]
binding = "MY_QUEUE"
queue = "my-queue"

[durable_objects]
bindings = [{ name = "CHAT_ROOM", class_name = "ChatRoom" }]

[[migrations]]
tag = "v1"
new_classes = ["ChatRoom"]
```

## Web Performance (Core Web Vitals)

```ts
// Edge middleware for performance headers
export default {
  async fetch(request: Request): Promise<Response> {
    const response = await fetch(request);
    const headers = new Headers(response.headers);

    // Cache static assets aggressively
    if (request.url.match(/\.(js|css|png|jpg|woff2)$/)) {
      headers.set('Cache-Control', 'public, max-age=31536000, immutable');
    }

    // Security headers
    headers.set('X-Frame-Options', 'DENY');
    headers.set('X-Content-Type-Options', 'nosniff');

    return new Response(response.body, { status: response.status, headers });
  },
};
```

## Sources
- cloudflare/agents-sdk
- cloudflare/building-ai-agent-on-cloudflare
- cloudflare/building-mcp-server-on-cloudflare
- cloudflare/durable-objects
- cloudflare/wrangler
- cloudflare/web-perf
