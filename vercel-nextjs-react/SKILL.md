---
name: vercel-nextjs-react
description: Vercel, Next.js, and React best practices — component patterns, caching strategies, App Router, SSR/SSG, deployment, and React Native. Auto-activates for Next.js, React, or Vercel projects.
origin: VoltAgent/awesome-agent-skills (vercel-labs)
tools: Read, Write, Edit, Bash
---

# Vercel / Next.js / React Skills

Official skills from the Vercel engineering team covering React, Next.js, and deployment.

## React Best Practices

### Component Patterns
```tsx
// Prefer composition over inheritance
function Card({ children, header }: { children: React.ReactNode; header: React.ReactNode }) {
  return (
    <div className="card">
      <div className="card-header">{header}</div>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Use forwardRef for reusable inputs
const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label, ...props }, ref) => (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  )
);
```

### State Management
- Use `useState` for local UI state
- Use `useReducer` for complex state transitions
- Use `useContext` sparingly — prefer prop drilling or state libs for perf-sensitive trees
- Co-locate state as close to where it's used as possible

### Performance
```tsx
// Memoize expensive renders
const ExpensiveList = React.memo(({ items }) => (
  <ul>{items.map(i => <li key={i.id}>{i.name}</li>)}</ul>
));

// Stable callbacks with useCallback
const handleClick = useCallback((id: string) => {
  setSelected(id);
}, []); // deps: only what changes

// Defer non-critical state updates
const [value, setValue] = useState('');
const deferredValue = useDeferredValue(value);
```

## Next.js Best Practices (App Router)

### Directory Structure
```
app/
  layout.tsx          # Root layout (persistent shell)
  page.tsx            # / route
  (auth)/             # Route group (no URL segment)
    login/page.tsx
  dashboard/
    layout.tsx        # Nested layout
    page.tsx          # /dashboard
    loading.tsx       # Suspense fallback
    error.tsx         # Error boundary
  api/
    route.ts          # API route handler
components/
  ui/                 # Reusable UI (client components)
  server/             # Server components
lib/
  db.ts
  auth.ts
```

### Server vs Client Components
```tsx
// Server Component (default) — no 'use client'
// Can: fetch data, access backend directly, use secrets
// Cannot: useState, useEffect, browser APIs, event handlers
async function ProductList() {
  const products = await db.query('SELECT * FROM products');
  return <ul>{products.map(p => <ProductCard key={p.id} product={p} />)}</ul>;
}

// Client Component — add 'use client' at top
'use client';
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### Data Fetching
```tsx
// Server Component — fetch with cache control
async function Page() {
  // Cached indefinitely (ISR-like)
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 } // revalidate every hour
  });

  // No cache (always fresh)
  const live = await fetch('https://api.example.com/live', {
    cache: 'no-store'
  });

  return <Dashboard data={await data.json()} />;
}
```

### Next.js Caching Strategy
```tsx
// unstable_cache for DB queries
import { unstable_cache } from 'next/cache';

const getUser = unstable_cache(
  async (id: string) => db.users.findUnique({ where: { id } }),
  ['user'],
  { revalidate: 60, tags: ['users'] }
);

// Revalidate on mutation
import { revalidateTag, revalidatePath } from 'next/cache';

async function updateUser(id: string, data: UserData) {
  await db.users.update({ where: { id }, data });
  revalidateTag('users');          // invalidate all 'users' cache
  revalidatePath('/dashboard');    // invalidate specific path
}
```

### Route Handlers (API)
```ts
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');
  const user = await db.users.findUnique({ where: { id } });
  return NextResponse.json(user);
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  const user = await db.users.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}
```

### Middleware
```ts
// middleware.ts (root)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token');
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/protected/:path*'],
};
```

## Vercel Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy (auto-detects framework)
vercel

# Deploy to production
vercel --prod

# Environment variables
vercel env add DATABASE_URL production
vercel env pull .env.local  # pull to local

# Preview deployments
# Every git push to non-main branch = automatic preview URL
```

### vercel.json
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api/$1" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [{ "key": "X-Content-Type-Options", "value": "nosniff" }]
    }
  ],
  "regions": ["iad1"]
}
```

## React Native (Vercel Labs)

```tsx
// Performance: use FlashList instead of FlatList
import { FlashList } from '@shopify/flash-list';
<FlashList
  data={items}
  renderItem={({ item }) => <Item data={item} />}
  estimatedItemSize={80}
/>

// Images: use expo-image for performance
import { Image } from 'expo-image';
<Image source={{ uri: url }} contentFit="cover" transition={200} />

// Navigation: avoid anonymous functions in navigate()
// BAD:  onPress={() => navigate('Detail', { id })}
// GOOD: const goToDetail = useCallback(() => navigate('Detail', { id }), [id]);
```

## Next.js Upgrade

```bash
# Upgrade Next.js
npx @next/codemod@canary upgrade latest

# Check for breaking changes
npx next info

# Run codemods for specific versions
npx @next/codemod built-in-next-font .
npx @next/codemod new-link .
```

## Sources
- vercel-labs/react-best-practices
- vercel-labs/next-best-practices
- vercel-labs/next-cache-components
- vercel-labs/composition-patterns
- vercel-labs/web-design-guidelines
- vercel-labs/next-upgrade
- vercel-labs/react-native-skills
