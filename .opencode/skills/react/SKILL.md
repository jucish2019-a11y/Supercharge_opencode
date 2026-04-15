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

## When to use me

Use this skill when:
- Building new React components or pages
- Refactoring class components to hooks
- Setting up state management for a feature
- Debugging React rendering or state issues
- Adding TypeScript types to a React codebase

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

## Patterns I follow

- Co-locate tests, stories, and styles with components
- One component per file, named exports, default export only at page level
- Custom hooks for reusable logic
- Error boundaries at route/page boundaries
- Suspense boundaries with meaningful fallbacks
- Keys from stable, unique IDs — never array indices for dynamic lists

## Anti-patterns I avoid

- Prop drilling beyond 2 levels (use context or state library)
- Storing derived state that can be computed
- Effects for synchronizing with external systems without cleanup
- `useEffect` for data fetching (use a data-fetching library)
- Inline object/array creation in JSX that causes re-renders

## Output structure

For new components:
```
ComponentName/
  ComponentName.tsx       # Component implementation
  ComponentName.test.tsx   # Tests
  ComponentName.stories.tsx # Stories (if Storybook used)
  index.ts                 # Re-export
```