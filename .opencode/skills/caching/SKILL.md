---
name: caching
description: Implement caching at every layer — browser caching, CDN, API response caching, React Query/SWR, Redis, and cache invalidation strategies
---

## What I do

I implement caching strategies across the full stack for fast, efficient applications:

- **Browser caching** — Cache-Control headers, service workers, ETags
- **CDN caching** — Vercel Edge, Cloudflare, cache key design
- **API response caching** — Next.js ISR, route handlers, revalidation
- **Client-side caching** — React Query, SWR, stale-while-revalidate
- **Server-side caching** — Redis, memoization, computation caching
- **Cache invalidation** — Revalidation, purge, tag-based, time-based

## When to use me

Use this skill when:
- Setting up caching for a new application
- Optimizing API response times
- Implementing CDN or edge caching
- Deciding what to cache and for how long
- Designing cache invalidation strategies
- Debugging stale data issues

## Caching layers

```
Request flow with caching at each layer:

Browser Request
    │
    ├── 1. Browser Cache      ← Cache-Control headers
    │      HIT → Return cached response
    │
    ├── 2. Service Worker     ← Offline cache, cache-first strategy
    │      HIT → Return cached response
    │
    ├── 3. CDN / Edge        ← Vercel Edge, Cloudflare
    │      HIT → Return from nearest PoP
    │
    ├── 4. Server Cache       ← Redis, in-memory
    │      HIT → Return cached computation
    │
    ├── 5. Application        ← Next.js ISR, route cache
    │      HIT → Return pre-rendered page
    │
    └── 6. Database           ← Query cache, materialized views
           HIT → Return cached query result
           MISS → Compute and cache at each layer
```

## Browser caching

### Cache-Control headers

```
Static assets (images, CSS, JS, fonts):
  Cache-Control: public, max-age=31536000, immutable
  - 1 year cache, content-hash in filename makes this safe
  - "immutable" tells browser to never revalidate

HTML pages:
  Cache-Control: no-cache
  - Always revalidate with server (ETag or Last-Modified)
  - Don't cache stale HTML

API responses (depends on data):
  Private user data:    Cache-Control: private, no-store
  Public, changes often: Cache-Control: public, max-age=60, s-maxage=300
  Public, changes rarely: Cache-Control: public, max-age=3600, s-maxage=86400
```

### Next.js static asset caching

```js
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/fonts/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=31536000, immutable' },
        ],
      },
      {
        source: '/_next/static/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=31536000, immutable' },
        ],
      },
    ];
  },
};
```

## Next.js caching

### Static Site Generation (SSG)

```tsx
// Cached at build time, served from CDN
async function Page() {
  const data = await fetch('https://api.example.com/data', {
    cache: 'force-cache',
  });
  return <DataDisplay data={data} />;
}
```

### Incremental Static Regeneration (ISR)

```tsx
// Cached, but revalidated in the background
async function Page() {
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 }, // Revalidate every hour
  });
  return <DataDisplay data={data} />;
}

// On-demand revalidation (after mutation)
import { revalidatePath, revalidateTag } from 'next/cache';

async function updateProject(formData: FormData) {
  'use server';
  await db.project.update({ ... });
  revalidatePath('/projects');       // Revalidate the projects page
  revalidateTag('projects');          // Revalidate all fetches with this tag
}
```

### fetch cache tags

```tsx
// Tag-based caching — invalidate by tag instead of URL
async function getProject(id: string) {
  const res = await fetch(`/api/projects/${id}`, {
    next: {
      tags: [`project-${id}`, 'projects'],
      revalidate: 3600,
    },
  });
  return res.json();
}

// Invalidate all project caches after a mutation
revalidateTag('projects');                  // All fetches tagged 'projects'
revalidateTag(`project-${projectId}`);       // Specific project fetches
```

### Route handler caching

```tsx
// app/api/projects/route.ts
export async function GET(request: Request) {
  const projects = await db.project.findMany();

  return new Response(JSON.stringify({ data: projects }), {
    headers: {
      'Cache-Control': 'public, s-maxage=60, stale-while-revalidate=300',
    },
  });
}
```

`stale-while-revalidate`: Serve stale data for up to 300s while fetching fresh data in the background. Users never wait.

## Client-side caching (React Query)

### Stale time vs cache time

```
staleTime:  How long before data is considered "stale" (background refetch)
  Default: 0 (always stale)
  Recommended: 30s - 5min for most data

gcTime:    How long before unused cache is garbage collected
  Default: 5 minutes
  Recommended: 5-30 minutes

Examples:
  User profile:    staleTime: 5min, gcTime: 30min
  Project list:   staleTime: 1min, gcTime: 10min
  Search results:  staleTime: 30s, gcTime: 5min
  Static content:  staleTime: Infinity, gcTime: 24h
```

### Prefetching

```tsx
// Prefetch data before the user navigates
import { queryClient } from '@/lib/query-client';

function ProjectListHover({ project }) {
  const handleMouseEnter = () => {
    queryClient.prefetchQuery({
      queryKey: ['projects', project.id],
      queryFn: () => fetch(`/api/projects/${project.id}`).then(r => r.json()),
      staleTime: 5 * 60 * 1000,
    });
  };

  return (
    <Link
      href={`/projects/${project.id}`}
      onMouseEnter={handleMouseEnter}
    >
      {project.name}
    </Link>
  );
}
```

## Server-side caching (Redis)

### When to use Redis

```
Use Redis when:
  - Same expensive query runs >10x per second
  - Computation takes >100ms and result is reusable
  - Rate limiting state needs to be shared across instances
  - Session storage for distributed servers
  - Real-time features (pub/sub, leaderboards)

Don't use Redis when:
  - Data changes every request (can't cache effectively)
  - Computation is fast (<10ms) — overhead > benefit
  - Database query cache is sufficient
  - Data size > 10MB per key (Redis is in-memory)
```

### Redis caching pattern

```ts
import { redis } from '@/lib/redis';

async function getProjectWithCache(id: string): Promise<Project> {
  const cacheKey = `project:${id}`;

  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Cache miss — fetch from DB
  const project = await db.project.findUnique({ where: { id } });
  if (!project) throw new NotFoundError('Project');

  // Store in cache (1 hour TTL)
  await redis.set(cacheKey, JSON.stringify(project), 'EX', 3600);

  return project;
}

// Invalidation — after mutation
async function updateProject(id: string, data: UpdateInput) {
  const project = await db.project.update({ where: { id }, data });
  await redis.del(`project:${id}`);
  return project;
}
```

### Cache-aside with stampede protection

```ts
import { redis } from '@/lib/redis';

const locks = new Map<string, Promise<any>>();

async function getOrFetch<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 3600
): Promise<T> {
  // Check cache
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  // Stampede protection — one request computes, others wait
  if (locks.has(key)) return locks.get(key)!;

  const promise = fetcher().then(async (data) => {
    await redis.set(key, JSON.stringify(data), 'EX', ttl);
    locks.delete(key);
    return data;
  });

  locks.set(key, promise);
  return promise;
}
```

## Cache invalidation strategies

### There are only two hard problems in CS: cache invalidation and naming things.

### Strategy 1: Time-based (TTL)

```
Simplest: set a TTL and let the cache expire
  - API responses: 60-300s
  - Database queries: 300-3600s
  - Static assets: 31536000s (1 year, content-hashed)
  - User profiles: 300-600s
  
Pros: Simple, predictable
Cons: Stale for up to TTL duration
Best for: Data that can tolerate short staleness
```

### Strategy 2: Event-based (purge on write)

```
When data changes, immediately invalidate the cache
  - After updating a project → delete cache key
  - After creating a task → invalidate project:tasks key
  - After user login → invalidate user:profile key
  - After any mutation → revalidatePath() or revalidateTag()

Pros: Always fresh
Cons: More complex, requires tracking all mutation points
Best for: Data that must be fresh (balances, counts, status)
```

### Strategy 3: Tag-based (grouped invalidation)

```
Tag related cache entries, invalidate by tag
  - All project queries tagged 'projects'
  - Invalidate tag 'projects' after any project mutation
  - Next.js: revalidateTag('projects')
  - Redis: maintain a tag→keys mapping, delete all keys for a tag

Pros: Efficient for group invalidation
Cons: Requires tag management infrastructure
Best for: Apps with many related cache entries
```

### Strategy 4: Stale-while-revalidate

```
Always serve from cache, refresh in background
  - Browser: Cache-Control: s-maxage=60, stale-while-revalidate=300
  - React Query: staleTime + refetchOnWindowFocus
  - SWR: built-in stale-while-revalidate

Pros: Zero latency for users, always improving
Cons: Data is always slightly stale
Best for: Most read-heavy APIs (default strategy)
```

### Choosing a strategy

```
Data that changes on every request?  → Don't cache
Data that changes every few minutes?  → TTL (60-300s) + SWR
Data that changes on user action?     → Event-based (purge on write)
Data that changes rarely?             → TTL (hours-days) + ISR
Related data that changes together?   → Tag-based

Most apps: SWR for reads + event-based invalidation for writes
```

## CDN caching

### Cache key design

```
CDN cache key = URL + query parameters + headers (vary)

Rules:
  - Don't vary on cookies for public content (breaks CDN caching)
  - Don't include user-specific data in CDN-cacheable URLs
  - Use /api/public/... for public CDN-cacheable endpoints
  - Use /api/user/... for authenticated, no-CDN endpoints

Vercel Edge:
  - Static pages: cached automatically at edge
  - ISR pages: cached at edge, revalidated in background
  - API routes: only cached if Cache-Control header is set
```

## Quality checklist

- [ ] Cache-Control headers set on all responses
- [ ] Static assets have immutable + 1-year max-age
- [ ] HTML pages use no-cache (always revalidate)
- [ ] API responses use stale-while-revalidate
- [ ] React Query has appropriate staleTime and gcTime
- [ ] Expensive server queries cached in Redis
- [ ] Cache invalidated on mutations (event-based)
- [ ] Next.js ISR with revalidation for semi-dynamic pages
- [ ] fetch() calls use next.tags for grouped invalidation
- [ ] No caching of authenticated/private data on CDN
- [ ] Stampede protection on high-traffic cache misses

## Anti-patterns I avoid

- Caching everything with the same TTL — different data needs different strategies
- Caching without an invalidation plan — stale data that you can't refresh
- Varying CDN cache on cookies — kills CDN hit rate
- No stale-while-revalidate — users wait for fresh data instead of getting stale + background refresh
- Cache-Control: no-store on public content — prevents all caching
- Redis for data that changes every request — overhead > benefit
- Forgetting to invalidate after mutations — users see stale data
- Caching authenticated user data on CDN — privacy violation