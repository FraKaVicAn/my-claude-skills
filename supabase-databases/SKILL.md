---
name: supabase-databases
description: Supabase, Neon serverless Postgres, ClickHouse analytics, and DuckDB — best practices for SQL, RLS, realtime, vector search, and data engineering. Auto-activates for database projects.
origin: VoltAgent/awesome-agent-skills (supabase, neon, clickhouse, duckdb)
tools: Read, Write, Edit, Bash
---

# Databases: Supabase / Neon / ClickHouse / DuckDB

## Supabase

### Setup
```bash
npm install @supabase/supabase-js
```

```ts
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js';
import type { Database } from './database.types';

export const supabase = createClient<Database>(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

// Server-side (bypasses RLS)
export const supabaseAdmin = createClient<Database>(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
```

### CRUD
```ts
// Select
const { data, error } = await supabase
  .from('posts')
  .select('id, title, author:users(name, avatar)')
  .eq('published', true)
  .order('created_at', { ascending: false })
  .limit(10);

// Insert
const { data, error } = await supabase
  .from('posts')
  .insert({ title: 'Hello', body: '...', user_id: user.id })
  .select()
  .single();

// Update
const { error } = await supabase
  .from('posts')
  .update({ published: true })
  .eq('id', postId);

// Delete
const { error } = await supabase
  .from('posts')
  .delete()
  .eq('id', postId);
```

### Row-Level Security (RLS)
```sql
-- Enable RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Users can only read published posts or their own
CREATE POLICY "read_posts" ON posts
  FOR SELECT USING (
    published = true OR auth.uid() = user_id
  );

-- Users can only insert their own posts
CREATE POLICY "insert_own_posts" ON posts
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Users can only update their own posts
CREATE POLICY "update_own_posts" ON posts
  FOR UPDATE USING (auth.uid() = user_id);
```

### Auth
```ts
// Sign up
const { data, error } = await supabase.auth.signUp({
  email, password,
  options: { data: { full_name: name } },
});

// Sign in
const { data, error } = await supabase.auth.signInWithPassword({ email, password });

// OAuth
await supabase.auth.signInWithOAuth({ provider: 'github' });

// Get current user
const { data: { user } } = await supabase.auth.getUser();

// Sign out
await supabase.auth.signOut();
```

### Realtime
```ts
const channel = supabase
  .channel('posts-changes')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'posts' },
    (payload) => console.log('Change:', payload)
  )
  .subscribe();

// Cleanup
supabase.removeChannel(channel);
```

### Vector Search (pgvector)
```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE documents (
  id bigserial PRIMARY KEY,
  content text,
  embedding vector(1536)
);

-- Similarity search
SELECT content, 1 - (embedding <=> $1) AS similarity
FROM documents
ORDER BY embedding <=> $1
LIMIT 5;

-- Index for performance
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);
```

## Neon Serverless Postgres

```bash
npm install @neondatabase/serverless
```

```ts
import { neon, neonConfig } from '@neondatabase/serverless';
import { Pool } from '@neondatabase/serverless';

// HTTP mode (serverless-friendly, no connection pooling needed)
const sql = neon(process.env.DATABASE_URL!);
const users = await sql`SELECT * FROM users WHERE id = ${userId}`;

// Pool mode (for many concurrent queries)
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const { rows } = await pool.query('SELECT * FROM users');

// WebSocket mode (for transactions)
neonConfig.webSocketConstructor = WebSocket;
const client = await pool.connect();
await client.query('BEGIN');
await client.query('INSERT INTO orders VALUES ($1, $2)', [orderId, total]);
await client.query('COMMIT');
client.release();
```

### Claimable Postgres (Neon)
```ts
// Provision a DB per user/tenant dynamically
const db = await neon.createDatabase({
  projectId: process.env.NEON_PROJECT_ID!,
  branchName: `user-${userId}`,
});
// Returns connection string for the new branch
```

## ClickHouse (Analytics)

```bash
npm install @clickhouse/client
```

```ts
import { createClient } from '@clickhouse/client';

const client = createClient({
  host: process.env.CLICKHOUSE_HOST,
  username: process.env.CLICKHOUSE_USER,
  password: process.env.CLICKHOUSE_PASSWORD,
  database: 'analytics',
});

// Insert events
await client.insert({
  table: 'events',
  values: [
    { timestamp: new Date(), user_id: '123', event: 'page_view', page: '/home' },
  ],
  format: 'JSONEachRow',
});

// Aggregate query
const result = await client.query({
  query: `
    SELECT
      toStartOfDay(timestamp) AS day,
      count() AS views,
      uniqExact(user_id) AS unique_users
    FROM events
    WHERE event = 'page_view'
      AND timestamp >= now() - INTERVAL 30 DAY
    GROUP BY day
    ORDER BY day
  `,
  format: 'JSONEachRow',
});

const rows = await result.json();
```

### ClickHouse Best Practices
- Use `MergeTree` engine family for all production tables
- Always specify `ORDER BY` (primary key) — determines sort order and query performance
- Use `ReplacingMergeTree` for deduplication, `SummingMergeTree` for aggregations
- Batch inserts: minimum 1000 rows per insert for efficiency
- Use `DateTime64` for sub-second timestamps
- Avoid `SELECT *` — specify columns explicitly

## DuckDB

```bash
pip install duckdb
# or
npm install duckdb-async
```

```python
import duckdb

# In-memory or file-based
con = duckdb.connect(':memory:')          # in-memory
con = duckdb.connect('analytics.duckdb') # persistent

# Query CSV, Parquet, JSON directly
con.execute("SELECT * FROM 'data.csv' LIMIT 10").fetchdf()
con.execute("SELECT * FROM 'data.parquet' WHERE amount > 1000").fetchdf()
con.execute("SELECT * FROM read_json_auto('data.json')").fetchdf()

# Query remote files
df = con.execute("""
  SELECT year, sum(sales) as total
  FROM 'https://example.com/sales.parquet'
  GROUP BY year
  ORDER BY year
""").fetchdf()

# Attach existing databases
con.execute("ATTACH 'mydb.duckdb' AS db")
con.execute("SELECT * FROM db.users")

# Install extensions
con.execute("INSTALL httpfs; LOAD httpfs")
con.execute("INSTALL spatial; LOAD spatial")
```

### DuckDB SQL (Friendly SQL)
```sql
-- Automatic column detection
SELECT * FROM 'folder/**/*.parquet';

-- UNPIVOT
UNPIVOT sales ON jan, feb, mar INTO NAME month VALUE amount;

-- PIVOT
PIVOT sales ON category USING sum(amount);

-- Struct access
SELECT data.user.name FROM events;

-- List operations
SELECT list_aggregate(scores, 'mean') FROM results;
```

## Sources
- supabase/postgres-best-practices
- neondatabase/neon-postgres
- neondatabase/claimable-postgres
- neondatabase/neon-postgres-egress-optimizer
- clickhouse/clickhouse-best-practices
- duckdb/query
- duckdb/read-file
- duckdb/install-duckdb
