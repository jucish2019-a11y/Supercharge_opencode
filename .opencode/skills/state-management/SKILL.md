---
name: state-management
description: Choose and implement the right state management for React — local state, context, Zustand, Jotai, React Query, and server state patterns that scale without over-engineering
---

## What I do

I design state management architectures that match the scale of the problem:

- **State classification** — Local vs. global vs. server vs. URL state
- **Local state patterns** — useState, useReducer, custom hooks
- **Context patterns** — When context is appropriate and when it isn't
- **Global state** — Zustand, Jotai for truly shared client state
- **Server state** — React Query / SWR for async data
- **URL state** — Search params as source of truth for filters, pagination, tabs
- **Form state** — Controlled vs uncontrolled, form libraries, validation

## When to use me

Use this skill when:
- Choosing a state management solution for a new project
- Refactoring prop drilling into proper state management
- Deciding between Zustand, Jotai, Redux, or Context
- Structuring async data fetching with React Query
- Managing complex form state
- Storing UI state (modals, filters, selected items) correctly

## The golden rule: use the lightest tool that works

```
State management complexity scale (use the LEFTMOST that solves your problem):

1. useState/useReducer        → Component-local state
2. URL search params          → Filters, pagination, active tab, sort
3. Context + useState        → Theme, auth, locale (read-heavy, rarely changes)
4. React Query / SWR          → Server data (API responses)
5. Zustand / Jotai           → Shared client state (toasts, modals, selections)
6. Redux                      → Never (unless existing codebase requires it)
```

## State classification

| State type | Source | Persisted? | Example | Tool |
|-----------|--------|-----------|---------|------|
| **Local** | Component | No | Form input, toggle, hover | useState |
| **Shared client** | Multiple components | No | Selected item, open modal | Zustand/Jotai |
| **Server** | API | Yes (remote) | User profile, projects list | React Query |
| **URL** | Browser address bar | Yes (bookmarkable) | Active tab, page, filter | useSearchParams |
| **Form** | User input | No (until submit) | Registration form | useState or form lib |
| **Persistent** | localStorage | Yes (local) | Theme, recent searches | Custom hook + localStorage |

**Never use global state for something that can be local.** If only one component needs the state, keep it there.

## Local state patterns

### useState for simple state

```tsx
function SearchBar() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

### useReducer for complex local state

```tsx
type State = {
  items: CartItem[];
  discount: number | null;
  shipping: Address | null;
};

type Action =
  | { type: 'add_item'; item: CartItem }
  | { type: 'remove_item'; id: string }
  | { type: 'apply_discount'; code: string }
  | { type: 'set_shipping'; address: Address };

function cartReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'add_item':
      return { ...state, items: [...state.items, action.item] };
    case 'remove_item':
      return { ...state, items: state.items.filter(i => i.id !== action.id) };
    case 'apply_discount':
      return { ...state, discount: action.code };
    case 'set_shipping':
      return { ...state, shipping: action.address };
  }
}
```

### Custom hooks for reusable logic

```tsx
function useToggle(initial = false): [boolean, () => void, () => void, () => void] {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  return [value, toggle, setTrue, setFalse];
}

// Usage
const [isOpen, toggle, open, close] = useToggle();
```

## Context — use sparingly

### When Context is appropriate

- Theme (light/dark) — changes rarely, read by many components
- Auth state (current user) — changes rarely, read by many
- Locale/i18n — changes rarely, read by many
- Feature flags — changes rarely, read by many

### When Context is NOT appropriate

- High-frequency updates (toasts, hover state, form inputs) — re-renders all consumers
- Frequently changing selections (selected row, current tab) — use URL state or Zustand
- Server data — use React Query

### Context pattern

```tsx
type Theme = 'light' | 'dark';

const ThemeContext = createContext<{
  theme: Theme;
  setTheme: (theme: Theme) => void;
}>(null!);

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>(
    () => (localStorage.getItem('theme') as Theme) ?? 'light'
  );

  useEffect(() => {
    localStorage.setItem('theme', theme);
    document.documentElement.dataset.theme = theme;
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

## Zustand — shared client state

Use when: multiple components need to read/write the same client-side state.

```tsx
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

type UiState = {
  selectedProjectId: string | null;
  isSidebarOpen: boolean;
  activeModal: string | null;
  toasts: Toast[];

  selectProject: (id: string) => void;
  toggleSidebar: () => void;
  openModal: (id: string) => void;
  closeModal: () => void;
  addToast: (toast: Omit<Toast, 'id'>) => void;
  removeToast: (id: string) => void;
};

export const useUiStore = create<UiState>()(
  devtools(
    (set) => ({
      selectedProjectId: null,
      isSidebarOpen: true,
      activeModal: null,
      toasts: [],

      selectProject: (id) => set({ selectedProjectId: id }),
      toggleSidebar: () => set((s) => ({ isSidebarOpen: !s.isSidebarOpen })),
      openModal: (id) => set({ activeModal: id }),
      closeModal: () => set({ activeModal: null }),
      addToast: (toast) =>
        set((s) => ({
          toasts: [...s.toasts, { ...toast, id: crypto.randomUUID() }],
        })),
      removeToast: (id) =>
        set((s) => ({ toasts: s.toasts.filter((t) => t.id !== id) })),
    }),
    { name: 'ui-store' }
  )
);
```

### Zustand best practices

- One store per domain (ui-store, cart-store, editor-store) — not one mega-store
- Use `devtools` middleware in development for debugging
- Use `persist` middleware only for state that should survive refresh (preferences)
- Selector pattern to avoid unnecessary re-renders:
```tsx
// ✅ Only re-renders when selectedProjectId changes
const projectId = useUiStore((s) => s.selectedProjectId);

// ❌ Re-renders on ANY store change
const { selectedProjectId } = useUiStore();
```

## React Query — server state

Use for: all data fetched from APIs. React Query handles caching, refetching, stale-while-revalidate, and pagination.

### Query (read data)

```tsx
import { useQuery, useQueryClient } from '@tanstack/react-query';

function useProject(id: string) {
  return useQuery({
    queryKey: ['projects', id],
    queryFn: () => fetch(`/api/projects/${id}`).then(r => r.json()),
    staleTime: 5 * 60 * 1000,     // 5 min before refetch
    gcTime: 30 * 60 * 1000,       // 30 min cache lifetime
    enabled: !!id,                 // Don't fetch if no ID
  });
}

// Usage
function ProjectDetail({ id }: { id: string }) {
  const { data: project, isLoading, error } = useProject(id);

  if (isLoading) return <Skeleton />;
  if (error) return <Error message={error.message} />;
  return <ProjectView project={project} />;
}
```

### Mutation (write data)

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useCreateProject() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateProjectInput) =>
      fetch('/api/projects', {
        method: 'POST',
        body: JSON.stringify(data),
      }).then(r => r.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['projects'] });
    },
  });
}

// Usage with optimistic update
function useUpdateProject() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: UpdateProjectInput) =>
      fetch(`/api/projects/${data.id}`, {
        method: 'PATCH',
        body: JSON.stringify(data),
      }).then(r => r.json()),
    onMutate: async (data) => {
      await queryClient.cancelQueries({ queryKey: ['projects', data.id] });
      const previous = queryClient.getQueryData(['projects', data.id]);
      queryClient.setQueryData(['projects', data.id], (old: Project) => ({
        ...old,
        ...data,
      }));
      return { previous };
    },
    onError: (_err, data, context) => {
      queryClient.setQueryData(['projects', data.id], context?.previous);
    },
    onSettled: (_data, _err, data) => {
      queryClient.invalidateQueries({ queryKey: ['projects', data.id] });
    },
  });
}
```

### Query key strategy

```ts
// Hierarchical keys — invalidating a parent invalidates all children
['projects']                      // All projects
['projects', id]                  // One project
['projects', id, 'tasks']        // Tasks for a project
['projects', id, 'tasks', taskId] // One task

// Invalidating all projects:
queryClient.invalidateQueries({ queryKey: ['projects'] });
// This also invalidates ['projects', id] and ['projects', id, 'tasks']
```

## URL state — the most underused pattern

Use URL search params for any state the user would want to bookmark, share, or navigate back to:

```tsx
import { useSearchParams, usePathname, useRouter } from 'next/navigation';

function useDataTable() {
  const [searchParams, setSearchParams] = useSearchParams();
  const pathname = usePathname();
  const router = useRouter();

  const page = Number(searchParams.get('page')) || 1;
  const sort = searchParams.get('sort') || 'created_at';
  const order = searchParams.get('order') || 'desc';
  const search = searchParams.get('q') || '';

  function setParams(params: Record<string, string>) {
    const next = new URLSearchParams(searchParams);
    Object.entries(params).forEach(([key, value]) => {
      if (value) next.set(key, value);
      else next.delete(key);
    });
    router.push(`${pathname}?${next.toString()}`);
  }

  return { page, sort, order, search, setParams };
}
```

What belongs in URL state:
- Active tab (`?tab=settings`)
- Page/pagination (`?page=2`)
- Sort order (`?sort=name&order=asc`)
- Search query (`?q=project`)
- Selected item (`?project=123`)
- Filter selections (`?status=active&type=featured`)

What does NOT belong in URL state:
- Modal open/closed (unless deep-linkable)
- Hover state
- Temporary toasts
- Form draft data

## Form state

### Simple forms — useState

```tsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <input value={password} onChange={e => setPassword(e.target.value)} type="password" />
    </form>
  );
}
```

### Complex forms — React Hook Form + Zod

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email'),
  role: z.enum(['admin', 'member', 'viewer']),
});

type FormValues = z.infer<typeof schema>;

function InviteMemberForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } =
    useForm<FormValues>({ resolver: zodResolver(schema) });

  const onSubmit = async (data: FormValues) => {
    await inviteMember(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}

      <input {...register('email')} type="email" />
      {errors.email && <span>{errors.email.message}</span>}

      <select {...register('role')}>
        <option value="admin">Admin</option>
        <option value="member">Member</option>
        <option value="viewer">Viewer</option>
      </select>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Inviting...' : 'Invite'}
      </button>
    </form>
  );
}
```

## Persistence patterns

```tsx
function usePersistedState<T>(key: string, defaultValue: T): [T, (value: T) => void] {
  const [state, setState] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : defaultValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(state));
  }, [key, state]);

  return [state, setState];
}

// Usage
const [recentSearches, setRecentSearches] = usePersistedState<string[]>('recent-searches', []);
```

## Decision checklist

- [ ] Only 1 component needs this state? → useState
- [ ] Should this be bookmarkable/shareable? → URL search params
- [ ] Is this read by many components, changes rarely? → Context
- [ ] Is this fetched from an API? → React Query
- [ ] Is this shared client state across components? → Zustand/Jotai
- [ ] No global state for strictly local concerns

## Anti-patterns I avoid

- Context for high-frequency updates — re-renders all consumers
- React Query without proper query keys — cache misses and duplicate requests
- Putting server data in Zustand — React Query handles caching, refetching, and staleness
- One massive global store — split by domain
- Selecting entire store instead of specific fields — causes unnecessary re-renders
- Not using URL state for filterable/sortable pages — lost on refresh, not shareable
- Storing form data in global state — form state is local until submitted
- Not invalidating queries after mutations — stale data shown to users