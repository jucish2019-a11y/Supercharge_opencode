---
name: nextjs
description: Build full-stack React applications with Next.js — App Router, Server Components, API routes, middleware, auth patterns, and production deployment
---

## What I do

I build Next.js applications using the App Router and modern patterns:

- **App Router** — File-system routing, layouts, loading states, error boundaries
- **Server Components** — RSC architecture, server vs client boundaries, data fetching
- **API routes** — Route handlers, server actions, middleware, request validation
- **Data fetching** — SSR, SSG, ISR, streaming, Suspense boundaries
- **Auth integration** — NextAuth.js, Clerk, Supabase Auth patterns
- **Performance** — Image optimization, font loading, code splitting, metadata
- **Deployment** — Vercel, Docker, standalone output, edge runtime

## When to use me

Use this skill when:
- Building a new Next.js application
- Migrating from Pages Router to App Router
- Implementing server components and client components correctly
- Setting up API routes or server actions
- Adding authentication to a Next.js app
- Optimizing Next.js performance and SEO
- Deploying a Next.js application

## How I work

1. **Discover the Next.js version and config** — Check `package.json`, `next.config.js`, existing routes, and middleware.
2. **Follow App Router conventions** — Use the file-system routing, colocation, and route group patterns.
3. **Make the server/client boundary explicit** — Every component is server by default. Mark client components only when needed.
4. **Choose the right rendering strategy** — SSR, SSG, ISR, or streaming based on the page's data needs.
5. **Implement data fetching at the component level** — Fetch where you need it, not in a top-level provider.
6. **Add loading and error states** — Use `loading.tsx` and `error.tsx` for every route segment.
7. **Optimize for production** — Images, fonts, metadata, bundle analysis.

## App Router project structure

```
app/
  layout.tsx            ← Root layout (HTML, body, providers)
  page.tsx             ← Home page (/)
  loading.tsx          ← Loading state for home
  error.tsx            ← Error boundary for home
  not-found.tsx        ← 404 page

  (auth)/              ← Route group (doesn't affect URL)
    login/
      page.tsx         ← /login
    register/
      page.tsx         ← /register
    layout.tsx         ← Auth layout (centered, no sidebar)

  (dashboard)/         ← Route group for authenticated pages
    layout.tsx         ← Dashboard layout (sidebar + header)
    projects/
      page.tsx         ← /projects
      [id]/
        page.tsx       ← /projects/:id
        loading.tsx
        error.tsx
      new/
        page.tsx       ← /projects/new

  api/
    health/
      route.ts         ← GET /api/health
    webhooks/
      route.ts         ← POST /api/webhooks

components/
  ui/                   ← Shared UI components (Button, Card, Input)
  features/             ← Feature-specific components
  providers.tsx         ← Client providers (theme, auth, toast)

lib/
  db.ts                ← Database client
  auth.ts              ← Auth configuration
  utils.ts             ← Utility functions
  validations.ts       ← Zod schemas

middleware.ts           ← Next.js middleware (auth, redirects)
next.config.js
```

## Server vs Client components

### Decision tree

```
Does the component need:
  ├── Browser APIs (window, document, localStorage)? → Client
  ├── Event handlers (onClick, onChange)? → Client
  ├── React hooks (useState, useEffect, useReducer)? → Client
  ├── Context providers (createContext)? → Client
  └── None of the above? → Server (default)
```

### Server Components (default)

```tsx
// app/projects/page.tsx — Server Component (default)
// Can directly access databases, file system, env vars
// Cannot use useState, useEffect, onClick, browser APIs

async function ProjectsPage() {
  const projects = await db.project.findMany();

  return (
    <div>
      {projects.map(project => (
        <ProjectCard key={project.id} project={project} />
      ))}
    </div>
  );
}
```

### Client Components (explicit)

```tsx
"use client";

// components/project-card.tsx — Client Component
// Can use hooks, event handlers, browser APIs
// Cannot directly access databases, file system

import { useState } from 'react';

function ProjectCard({ project }) {
  const [isHovered, setIsHovered] = useState(false);

  return (
    <div
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <h3>{project.name}</h3>
      <button onClick={() => archiveProject(project.id)}>
        Archive
      </button>
    </div>
  );
}
```

### The boundary rule

Server components can render client components. Client components CANNOT render server components as children (except through `children` prop).

```tsx
// ✅ Server renders Client
// layout.tsx (Server)
function Layout({ children }) {
  return (
    <div>
      <Sidebar />        {/* Client component — fine */}
      {children}          {/* Slot — can be Server or Client */}
    </div>
  );
}

// ✅ Client receives Server children via slot
// sidebar.tsx (Client)
"use client";
function Sidebar({ children }) {
  return (
    <aside>
      <nav>...</nav>
      {children}          {/* This CAN be a Server component */}
    </aside>
  );
}

// ❌ Client cannot import and render Server directly
"use client";
function BadClient() {
  return (
    <div>
      <ServerComponent />  {/* ❌ This won't work */}
    </div>
  );
}
```

## Data fetching strategies

| Strategy | When | How | Cache |
|----------|------|-----|-------|
| **SSR** | Dynamic data, personalized pages | `fetch()` in Server Component, no cache | `cache: 'no-store'` |
| **SSG** | Static content, marketing pages | `fetch()` in Server Component with cache | `cache: 'force-cache'` |
| **ISR** | Semi-dynamic, updated periodically | `revalidate` in fetch or route config | `next: { revalidate: 3600 }` |
| **Streaming** | Slow data, progressive rendering | Suspense + async Server Components | Per-request |
| **Client** | Real-time, user-specific, interactive | SWR/React Query in Client Components | Browser |

### Server Component data fetching

```tsx
// SSR — fresh data every request
async function Page() {
  const res = await fetch('https://api.example.com/data', {
    cache: 'no-store'
  });
  const data = await res.json();
  return <DataDisplay data={data} />;
}

// SSG — cached at build time
async function Page() {
  const res = await fetch('https://api.example.com/data', {
    cache: 'force-cache'
  });
  const data = await res.json();
  return <DataDisplay data={data} />;
}

// ISR — revalidate every hour
async function Page() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 }
  });
  const data = await res.json();
  return <DataDisplay data={data} />;
}

// Route-level revalidation
export const revalidate = 3600; // Revalidate this page every hour
```

### Streaming with Suspense

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* Fast content renders immediately */}
      <QuickStats />

      {/* Slow content streams in */}
      <Suspense fallback={<ChartSkeleton />}>
        <AnalyticsChart />
      </Suspense>

      <Suspense fallback={<ListSkeleton />}>
        <RecentActivity />
      </Suspense>
    </div>
  );
}
```

## Server Actions

Server Actions are the idiomatic way to handle form submissions and mutations in Next.js:

```tsx
// app/projects/new/action.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const createProjectSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),
});

export async function createProject(formData: FormData) {
  const validated = createProjectSchema.parse({
    name: formData.get('name'),
    description: formData.get('description'),
  });

  const project = await db.project.create({ data: validated });

  revalidatePath('/projects');
  redirect(`/projects/${project.id}`);
}
```

```tsx
// app/projects/new/page.tsx
import { createProject } from './action';

function NewProjectPage() {
  return (
    <form action={createProject}>
      <input name="name" type="text" required />
      <textarea name="description" />
      <button type="submit">Create project</button>
    </form>
  );
}
```

### Server Action rules
- Always validate input with Zod or similar — never trust FormData
- Use `revalidatePath` or `revalidateTag` to update cached data after mutations
- Use `redirect` for post-mutation navigation
- Return error objects for validation failures — handle in the form
- For optimistic updates, pair with `useOptimistic` hook

## API Route Handlers

```tsx
// app/api/webhooks/stripe/route.ts
import { headers } from 'next/headers';
import { stripe } from '@/lib/stripe';

export async function POST(request: Request) {
  const body = await request.text();
  const signature = headers().get('stripe-signature');

  try {
    const event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET
    );

    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutComplete(event.data.object);
        break;
    }

    return Response.json({ received: true });
  } catch (err) {
    return Response.json({ error: 'Invalid signature' }, { status: 400 });
  }
}
```

## Middleware

```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Auth check
  const token = request.cookies.get('session-token');

  if (!token && !request.nextUrl.pathname.startsWith('/login')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Add security headers
  const response = NextResponse.next();
  response.headers.set('x-frame-options', 'DENY');
  response.headers.set('x-content-type-options', 'nosniff');

  return response;
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|api/health).*)',
  ],
};
```

Middleware rules:
- Runs on the Edge runtime — no Node.js APIs (no fs, no db directly)
- Keep it fast — it runs on every matched request
- Use for: auth redirects, geolocation, A/B testing, security headers
- DON'T use for: complex business logic, database queries

## Layouts and templates

### Root layout

```tsx
// app/layout.tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import { ThemeProvider } from '@/components/providers';

const inter = Inter({ subsets: ['latin'], display: 'swap' });

export const metadata: Metadata = {
  title: { default: 'App Name', template: '%s | App Name' },
  description: 'App description',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={inter.className}>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

### Nested layouts

```tsx
// app/(dashboard)/layout.tsx
import { Sidebar } from '@/components/sidebar';
import { Header } from '@/components/header';

export default function DashboardLayout({ children }) {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Header />
        <main className="flex-1 overflow-auto p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

### Loading states

```tsx
// app/projects/loading.tsx
// Automatically wraps the page in a Suspense boundary
function ProjectsLoading() {
  return (
    <div className="grid grid-cols-3 gap-6">
      {Array.from({ length: 6 }).map((_, i) => (
        <div key={i} className="h-48 rounded-lg bg-neutral-100 animate-pulse" />
      ))}
    </div>
  );
}
```

### Error boundaries

```tsx
// app/projects/error.tsx
'use client';

function ProjectsError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="flex flex-col items-center gap-4 py-20">
      <h2 className="text-xl font-semibold">Something went wrong</h2>
      <p className="text-neutral-500">{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

## Image and font optimization

```tsx
import Image from 'next/image';
import { Inter } from 'next/font/google';

// Fonts — self-hosted, no CLS, optimized
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

// Images — automatic sizing, lazy loading, blur placeholder
function ProfileCard({ user }) {
  return (
    <div className="flex items-center gap-3">
      <Image
        src={user.avatar}
        alt={`${user.name}'s avatar`}
        width={40}
        height={40}
        className="rounded-full"
      />
      <span>{user.name}</span>
    </div>
  );
}
```

Image rules:
- Always provide `width` and `height` (or `fill` for responsive)
- Always provide meaningful `alt` text
- Use `placeholder="blur"` with `blurDataURL` for local images
- For external images, add domains to `next.config.js` `images.remotePatterns`
- Use `priority` on above-the-fold images (disables lazy loading)

## Metadata and SEO

```tsx
// app/projects/[id]/page.tsx
import type { Metadata } from 'next';

// Static metadata
export const metadata: Metadata = {
  title: 'Projects',
  description: 'Manage your projects',
};

// Dynamic metadata
export async function generateMetadata({ params }): Promise<Metadata> {
  const project = await db.project.findUnique({ where: { id: params.id } });
  return {
    title: project.name,
    description: project.description,
    openGraph: {
      title: project.name,
      description: project.description,
      images: [{ url: project.coverImage, width: 1200, height: 630 }],
    },
  };
}
```

## Performance checklist

- [ ] Server Components by default, Client Components only where needed
- [ ] Suspense boundaries for slow data fetches
- [ ] Images use `next/image` with proper sizing
- [ ] Fonts use `next/font` (Google or local)
- [ ] No unnecessary Client Providers wrapping Server content
- [ ] `loading.tsx` for every route segment with data fetching
- [ ] `error.tsx` for error recovery
- [ ] Metadata defined for every page
- [ ] `next.config.js` has proper image domains configured
- [ ] Bundle analyzed for large client-side dependencies

## Anti-patterns I avoid

- Making everything a Client Component — defeats the purpose of RSC
- Fetching data in a top-level layout and drilling it down — fetch at the point of use
- Using `getServerSideProps` / `getStaticProps` (Pages Router API in App Router)
- Putting database queries in Client Components — use Server Components or API routes
- Missing `loading.tsx` — users see a blank page during data fetching
- Missing `error.tsx` — unhandled errors crash the app
- Not using `revalidatePath` after mutations — stale data shown
- Ignoring the middleware matcher — running middleware on static assets
- Using `next/router` in App Router — use `next/navigation` instead
- Importing server-only code in client bundles — use `server-only` package