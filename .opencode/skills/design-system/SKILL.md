---
name: design-system
description: Build design token systems, component libraries, and style guides for consistent UI at scale
---

## What I do

I build design systems that scale:

- **Design tokens** — Colors, typography, spacing, shadows as structured data
- **Token pipelines** — Figma → tokens → CSS variables → code
- **Component libraries** — Reusable, accessible, composable components
- **Documentation** — Storybook, usage examples, design principles
- **Versioning** — Semantic versioning, migration guides, deprecation

## When to use me

Use this skill when:
- Starting a new design system from scratch
- Scaling an existing component library
- Converting design tokens from Figma to code
- Documenting components and patterns
- Planning a design system migration
- Setting up token-to-code automation

## Token pipeline

### Token structure

```json
{
  "color": {
    "base": {
      "white": { "value": "#FFFFFF" },
      "black": { "value": "#000000" }
    },
    "brand": {
      "50": { "value": "#eff6ff" },
      "100": { "value": "#dbeafe" },
      "500": { "value": "#3b82f6" },
      "600": { "value": "#2563eb" },
      "900": { "value": "#1e3a8a" }
    },
    "semantic": {
      "background": { "value": "{color.base.white}" },
      "text": { "value": "{color.base.black}" },
      "primary": { "value": "{color.brand.500}" }
    }
  },
  "spacing": {
    "xs": { "value": "0.25rem" },
    "sm": { "value": "0.5rem" },
    "md": { "value": "1rem" },
    "lg": { "value": "1.5rem" },
    "xl": { "value": "2rem" }
  },
  "fontSize": {
    "xs": { "value": "0.75rem" },
    "sm": { "value": "0.875rem" },
    "base": { "value": "1rem" },
    "lg": { "value": "1.125rem" },
    "xl": { "value": "1.25rem" },
    "2xl": { "value": "1.5rem" }
  }
}
```

### Token-to-code pipeline

```bash
# Tools: Style Dictionary, Token Studio
# 1. Designers maintain tokens in Figma (Token Studio)
# 2. Export tokens as JSON
# 3. Transform to platform outputs
# 4. Commit to repo

# style-dictionary.config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'src/styles/',
      files: [{
        destination: 'tokens.css',
        format: 'css/variables',
      }],
    },
    ts: {
      transformGroup: 'js',
      buildPath: 'src/tokens/',
      files: [{
        destination: 'index.ts',
        format: 'javascript/es6',
      }],
    },
  },
};
```

## Component API design

### Compound components

```tsx
// Card compound component pattern
interface CardProps {
  children: ReactNode;
  className?: string;
}

function Card({ children, className }: CardProps) {
  return (
    <div className={cn('rounded-lg border bg-card text-card-foreground shadow-sm', className)}>
      {children}
    </div>
  );
}

function CardHeader({ children, className }: CardProps) {
  return <div className={cn('flex flex-col space-y-1.5 p-6', className)}>{children}</div>;
}

function CardTitle({ children, className }: CardProps) {
  return <h3 className={cn('text-2xl font-semibold leading-none tracking-tight', className)}>{children}</h3>;
}

function CardDescription({ children, className }: CardProps) {
  return <p className={cn('text-sm text-muted-foreground', className)}>{children}</p>;
}

function CardContent({ children, className }: CardProps) {
  return <div className={cn('p-6 pt-0', className)}>{children}</div>;
}

function CardFooter({ children, className }: CardProps) {
  return <div className={cn('flex items-center p-6 pt-0', className)}>{children}</div>;
}

Card.Header = CardHeader;
Card.Title = CardTitle;
Card.Description = CardDescription;
Card.Content = CardContent;
Card.Footer = CardFooter;

// Usage
<Card>
  <Card.Header>
    <Card.Title>Card Title</Card.Title>
    <Card.Description>Card description</Card.Description>
  </Card.Header>
  <Card.Content>Content here</Card.Content>
  <Card.Footer>
    <Button>Action</Button>
  </Card.Footer>
</Card>
```

## Design system repo structure

```
design-system/
├── packages/
│   ├── tokens/           # Design tokens (JSON → CSS/TS)
│   ├── components/       # React/Vue/Angular components
│   ├── icons/            # Icon library
│   └── utils/            # Shared utilities
├── apps/
│   └── storybook/        # Documentation and playground
├── tools/
│   ├── token-transformer/
│   └── figma-plugin/
└── package.json
```

## Versioning and migration

```
Semantic versioning for design systems:
├── MAJOR: Breaking changes (token renames, component API changes)
├── MINOR: New features (new components, new tokens)
└── PATCH: Bug fixes (color corrections, spacing fixes)

Migration strategy:
1. Deprecate old API with warnings
2. Provide codemod scripts for automated migration
3. Document breaking changes in CHANGELOG
4. Support old API for one major version (deprecation period)
```

## Quality checklist

- [ ] Design tokens cover all visual properties (colors, spacing, typography, shadows)
- [ ] Tokens have semantic naming (primary, not blue-500)
- [ ] Token pipeline automated (Figma → code)
- [ ] Components are accessible (keyboard, screen reader, focus)
- [ ] Components are composable (compound component pattern)
- [ ] API surface is minimal and intentional
- [ ] Documentation includes usage examples and props
- [ ] Versioning strategy documented
- [ ] Migration guides for breaking changes
- [ ] Visual regression testing in place

## Anti-patterns I avoid

- Hardcoding values instead of using tokens
- One-off components that duplicate existing patterns
- Breaking changes without deprecation warnings
- Not documenting component APIs
- Components that are too rigid (not composable)
- Design tokens that are too granular or too sparse
- Not versioning the design system independently
- Ignoring accessibility in component design
- Not testing components across themes
- Design and code getting out of sync