---
name: component-composition
description: Design and build reusable, composable, accessible UI components from atoms to full pages
---

## What I do

I design and implement UI components that are reusable, composable, and accessible:

- Break down pages into atomic components (atoms, molecules, organisms)
- Define component APIs (props, slots, variants)
- Implement composition patterns that scale
- Ensure every component is accessible and has all required states

## When to use me

Use this skill when:
- Building a new UI component or component library
- Breaking down a complex page into smaller components
- Making existing components more reusable and composable
- Designing component APIs that other developers can use

## How I work

1. **Deconstruct the UI** — Break the page into components following atomic design:
   - **Atoms**: Button, Input, Badge, Avatar, Icon, Spinner
   - **Molecules**: SearchBar (Input + Icon), FormField (Label + Input + Error), MenuItem (Icon + Text)
   - **Organisms**: Header (Logo + Nav + UserMenu), Sidebar (NavSection[]), DataTable
   - **Pages**: Composed from organisms and molecules

2. **Define the component API** — For each component, define:
   - **Props** (inputs): What data/config does it accept?
   - **Slots** (children): What content can be injected?
   - **Variants**: What visual variants? (primary, secondary, ghost, danger)
   - **Sizes**: What sizes? (sm, md, lg)
   - **States**: default, hover, focus, active, disabled, loading, error
   - **Events**: What actions does it emit?

3. **Implement composition over configuration** — Prefer slot-based composition over prop-based configuration:

   Good (composable):
   ```html
   <Card>
     <CardHeader>
       <Avatar src="..." />
       <CardTitle>Name</CardTitle>
     </CardHeader>
     <CardContent>Body</CardContent>
     <CardFooter>
       <Button>Action</Button>
     </CardFooter>
   </Card>
   ```

   Avoid (over-configured):
   ```html
   <Card avatar="..." title="Name" body="Body" action="Action" />
   ```

4. **Implement all required states** — Every component must handle:
   - Default state
   - Hover state (interactive elements)
   - Focus state (visible focus ring for keyboard users)
   - Active/pressed state
   - Disabled state
   - Loading state (if async)
   - Error state (if applicable)
   - Empty state (if data-driven)

5. **Make it accessible** — Follow WAI-ARIA patterns for the component type. Use semantic HTML. Support keyboard navigation.

6. **Document the component** — Show usage examples, all variants, and all states.

## Component file structure

```
components/
  Button/
    Button.tsx        -- Implementation
    Button.module.css -- Styles
    Button.test.tsx   -- Tests
    Button.stories.tsx -- Variants showcase
    index.ts          -- Export
```

## API design guidelines

- **Fewer props is better** — If a component needs >8 props, consider splitting it
- **Use the platform** — Pass through native HTML attributes, don't reinvent them
- **Sensible defaults** — Components should look good with zero configuration
- **Escape hatches** — Provide `className` or `style` overrides for edge cases
- **Type safety** — Props should be typed with discriminated unions for mutually exclusive variants
- **Children > props** — Use slots/children for content, props for configuration

## Guidelines

- Components should do one thing well
- Prefer composition over props for flexibility
- Every interactive element needs hover, focus, active, and disabled states
- Always use semantic HTML elements (`button` not `div` with click handler)
- Test components in isolation, then in composition
- Document every variant and state