---
name: react
description: Build React components and applications following modern best practices, patterns, and conventions
---

## What I do

I build React applications using current best practices:

- **Component architecture** — Functional components, hooks, composition over inheritance
- **State management** — Local state, context, reducers; choose the right tool for the scale
- **Performance** — Memoization, lazy loading, virtualization, avoiding unnecessary re-renders
- **TypeScript** — Proper typing for props, state, events, refs, and generics
- **Testing** — Component tests, hook tests, integration tests with appropriate tools
- **Server Components** — When to use 'use client', data fetching patterns
- **Concurrent features** — useTransition, useDeferredValue, Suspense

## When to use me

Use this skill when:
- Building new React components or pages
- Refactoring class components to hooks
- Setting up state management for a feature
- Debugging React rendering or state issues
- Adding TypeScript types to a React codebase
- Optimizing React performance

## How I work

1. **Discover the stack** — Check package.json for React version, router, state library, styling approach, test framework. Read existing components to understand patterns.
2. **Follow existing conventions** — Match naming, file structure, export style, and import order used in the project.
3. **Choose the right pattern**:
   - Simple local state → `useState`
   - Complex state logic → `useReducer`
   - Shared state across a few components → Context
   - Global async state → Data-fetching library (React Query, SWR) if available
   - Complex global state → Zustand/Redux/Jotai if already in the project
4. **Write the component** — Functional component, typed props, extracted hooks for logic, colocation principle.
5. **Optimize if needed** — `useMemo`/`useCallback` only when profiling shows a problem, `React.memo` for leaf components that re-render often.
6. **Add tests** — Test behavior, not implementation. Prefer testing user interactions over assertion on internal state.

## Hook patterns

### useState with complex types

```tsx
interface User {
  id: string;
  name: string;
  email: string;
}

function UserForm() {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const updateField = <K extends keyof User>(field: K, value: User[K]) => {
    setUser(prev => prev ? { ...prev, [field]: value } : null);
  };

  return (
    <form>
      <input
        value={user?.name ?? ''}
        onChange={e => updateField('name', e.target.value)}
      />
    </form>
  );
}
```

### useEffect rules and patterns

```tsx
// GOOD: Dependencies are complete
useEffect(() => {
  const controller = new AbortController();
  fetchData(id, { signal: controller.signal });
  return () => controller.abort();
}, [id]); // id is in dependency array

// BAD: Missing dependencies
useEffect(() => {
  fetchData(id); // ESLint will warn: id is missing from deps
}, []);

// GOOD: Split effects by concern
useEffect(() => {
  // Data fetching
  fetchData(id);
}, [id]);

useEffect(() => {
  // DOM manipulation
  document.title = title;
}, [title]);

// GOOD: useLayoutEffect for DOM measurements
useLayoutEffect(() => {
  const height = ref.current?.getBoundingClientRect().height;
  setMeasuredHeight(height);
}, [children]);

// GOOD: Custom hook for data fetching
function useUser(id: string) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setIsLoading(true);
        const data = await fetch(`/api/users/${id}`).then(r => r.json());
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err as Error);
      } finally {
        if (!cancelled) setIsLoading(false);
      }
    }

    fetchUser();
    return () => { cancelled = true; };
  }, [id]);

  return { user, isLoading, error };
}
```

## Server Components

### When to use 'use client'

```
'use client' needed?
├── Uses browser APIs (window, document, localStorage)?
│   └── YES → 'use client'
├── Has event handlers (onClick, onSubmit)?
│   └── YES → 'use client'
├── Uses React hooks (useState, useEffect)?
│   └── YES → 'use client'
├── Uses context?
│   └── YES → 'use client'
├── Just renders data passed as props?
│   └── NO → Server Component (default)
├── Fetches data from API?
│   └── NO → Server Component (fetch directly)
└── Just JSX with no interactivity?
    └── NO → Server Component
```

### Server Component pattern

```tsx
// This is a Server Component by default in Next.js App Router
async function UserProfile({ userId }: { userId: string }) {
  const user = await db.user.findUnique({ where: { id: userId } });

  if (!user) return <NotFound />;

  return (
    <div>
      <h1>{user.name}</h1>
      <ClientFollowButton userId={userId} />
    </div>
  );
}

// Client component for interactivity
'use client';

function ClientFollowButton({ userId }: { userId: string }) {
  const [isFollowing, setIsFollowing] = useState(false);

  const handleFollow = async () => {
    await fetch('/api/follow', { method: 'POST', body: JSON.stringify({ userId }) });
    setIsFollowing(true);
  };

  return (
    <button onClick={handleFollow}>
      {isFollowing ? 'Following' : 'Follow'}
    </button>
  );
}
```

## Performance patterns

### Memoization

```tsx
import { memo, useMemo, useCallback } from 'react';

// Only memoize when profiling shows a benefit
const ExpensiveList = memo(function ExpensiveList({ items, onSelect }: ListProps) {
  return (
    <ul>
      {items.map(item => (
        <ListItem key={item.id} item={item} onSelect={onSelect} />
      ))}
    </ul>
  );
});

function Parent() {
  const [items, setItems] = useState<Item[]>([]);
  const [filter, setFilter] = useState('');

  // Memoize expensive computation
  const filteredItems = useMemo(() => {
    return items.filter(item => item.name.includes(filter));
  }, [items, filter]);

  // Memoize callback to prevent child re-renders
  const handleSelect = useCallback((id: string) => {
    setItems(prev => prev.map(item =>
      item.id === id ? { ...item, selected: true } : item
    ));
  }, []);

  return <ExpensiveList items={filteredItems} onSelect={handleSelect} />;
}
```

### Code splitting

```tsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));
const AdminPanel = lazy(() => import('./AdminPanel'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>

      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart data={data} />
        </Suspense>
      )}

      <Suspense fallback={<AdminSkeleton />}>
        <AdminPanel />
      </Suspense>
    </div>
  );
}
```

### Error boundaries

```tsx
import { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <ErrorFallback error={this.state.error} />;
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorPage />}>
  <App />
</ErrorBoundary>
```

## Concurrent features

```tsx
import { useTransition, useDeferredValue } from 'react';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const deferredQuery = useDeferredValue(query);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    // Urgent update: show input immediately
    setQuery(value);
    // Transition: update results can be interrupted
    startTransition(() => {
      setSearchQuery(value);
    });
  };

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <Results query={deferredQuery} />
    </div>
  );
}
```

## Patterns I follow

- Co-locate tests, stories, and styles with components
- One component per file, named exports, default export only at page level
- Custom hooks for reusable logic
- Error boundaries at route/page boundaries
- Suspense boundaries with meaningful fallbacks
- Keys from stable, unique IDs — never array indices for dynamic lists
- Server Components by default, 'use client' only when needed

## Anti-patterns I avoid

- Prop drilling beyond 2 levels (use context or state library)
- Storing derived state that can be computed
- Effects for synchronizing with external systems without cleanup
- `useEffect` for data fetching (use a data-fetching library like React Query)
- Inline object/array creation in JSX that causes re-renders
- Not using keys or using array indices as keys for dynamic lists
- Using `useMemo`/`useCallback` for everything — only when profiling shows benefit
- Mixing server and client logic in the same component
- Not handling loading and error states
- Mutating state directly instead of using setState