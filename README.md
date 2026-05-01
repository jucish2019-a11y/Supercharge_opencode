# Supercharging OpenCode

A library of **82 specialized skills** covering design, engineering, infrastructure, and process, so your AI assistant makes decisions it can defend.

**[View the full showcase →](https://jucish2019-a11y.github.io/Supercharge_opencode/)**

---

## What is this?

Every AI code assistant can write working code. But without domain knowledge, every decision defaults to the most common pattern. Those convergent defaults compound. The result looks functional but indistinguishable from every other AI output.

This skill library gives [opencode](https://github.com/anomalyco/opencode) structured, theory-backed knowledge across 15 categories. From establishing design purpose before choosing colors, to deploying with rollback strategies, to stripping AI tells from your visual output.

Each skill doesn't just tell you *what* to do. It explains *why*: the principle behind the recommendation.

---

## The 15 Categories

| Category | Skills | What it covers |
|----------|--------|----------------|
| **Design Foundations & Visual Theory** | 6 | Purpose, composition, proportions, color science, auditing, AI tell detection |
| **Typography & Color** | 3 | Font pairing, type scales, mood-to-hue palettes, dark mode |
| **Layout & Spacing** | 2 | Gestalt grouping, responsive breakpoints |
| **Design Systems & Tokens** | 3 | Token architecture, component libraries, design specs |
| **Interface & User Experience** | 7 | End-to-end UI design, UX patterns, accessibility, landing pages |
| **Visual Assets** | 3 | Icons, illustration, data visualization |
| **Motion & Animation** | 3 | Micro-interactions, choreography, motion systems |
| **Component Architecture** | 6 | React, Next.js, CSS, state management, mobile |
| **TypeScript & Languages** | 3 | Type architecture, type safety, Python |
| **API & Backend** | 5 | REST/GraphQL design, Node.js backends, error handling |
| **Data & Databases** | 5 | Schema design, validation, Prisma, migrations, SQL |
| **Integration & Feature Services** | 11 | Auth, payments, email, uploads, search, real-time, caching, AI, i18n |
| **Infrastructure & DevOps** | 6 | Docker, CI/CD, deployment, IaC, environments, monitoring |
| **Version Control & Release** | 6 | Git commits, branching, PRs, code review, changelogs, releases |
| **Quality & Performance** | 8 | Testing, debugging, security, troubleshooting, refactoring, deps |

---

## Two Modes

Every design skill works in two modes:

**Checker mode**: Audit existing work. Run a 10-dimension review: purpose, typography, proportions, composition, hierarchy, color, semantic HTML, motion, responsiveness, and design identity. Every finding names the principle it violates and explains why it matters.

**Applier mode**: Build from scratch. Follow gated phases: foundations → type system → color system → proportions → composition → build → audit. Each phase produces decisions the next phase depends on.

---

## Key Principles

These are the core ideas that run through the library:

- **Three layers**: Purpose, medium, and aesthetics must reinforce each other. Constraints are design opportunities, not problems.
- **Intentional proportions**: Dimensions should relate through named ratios (1.25, 1.333, 1.5), not arbitrary pixel values.
- **Composition over decoration**: One dominant element. Not everything gets a card. White space is the most powerful hierarchy tool.
- **Mood drives hue**: Color starts from the feeling the product should communicate, not from "what looks nice."
- **Motion is information**: Remove an animation. If you lost information, it was functional. If not, it was decoration.
- **Eight interaction states**: Every element needs default, hover, focus, active, disabled, loading, error, and success.
- **Strip AI tells**: Default fonts, cyan-on-dark, identical card grids, centering disease, equal spacing, glassmorphism, neon accents, and uniform animations are signals. Replace them with defensible choices.

---

## All 82 Skills

### Design Foundations & Visual Theory
- **design-foundations**: Purpose, audience, aesthetic direction
- **composition**: Dominance, rhythm, texture, contrast, eye guidance
- **proportions**: Intentional type and space ratio systems
- **design-audit**: 10-dimension principle-based review
- **strip-ai-tells**: Detect and replace AI-generated design defaults
- **visual-hierarchy**: *(redirects to composition, proportions, spatial-design)*

### Typography & Color
- **typography**: Font pairing, modular scales, vertical rhythm
- **color-palette**: Mood-to-hue semantic color systems, accessibility
- **dark-mode**: Dual-theme contrast and elevation

### Layout & Spacing
- **spatial-design**: Gestalt grouping, density, containment
- **responsive-layout**: Adaptive layouts across all breakpoints

### Design Systems & Tokens
- **design-system**: Token systems and component libraries
- **design-tokens**: Scalable token-to-code pipelines
- **design-spec**: Pixel-perfect specs with all states

### Interface & User Experience
- **ui-design**: End-to-end page design workflow (orchestrator)
- **ux-patterns**: Navigation, forms, feedback patterns
- **interaction-design**: Drag, gestures, command palette, keyboard
- **accessibility**: WCAG compliance, ARIA, keyboard navigation
- **landing-page**: High-converting pages with narrative arc
- **content-design**: UX writing, microcopy, tone of voice
- **visual-qa**: Pixel-level spacing and alignment review

### Visual Assets
- **iconography**: Consistent icon system design
- **illustration-direction**: Cohesive illustration style guides
- **data-visualization**: Accurate, readable charts and dashboards

### Motion & Animation
- **micro-interactions**: Hover, loading, transition animation states
- **animation-choreography**: Multi-element page transitions
- **motion-system**: Easing, duration, choreography systems

### Component Architecture
- **react**: Modern React best practices
- **nextjs**: App Router, Server Components, production
- **component-composition**: Reusable, composable, accessible components
- **css-styling**: Tailwind, CSS modules, modern CSS
- **state-management**: Scalable React state patterns
- **mobile-development**: React Native cross-platform apps

### TypeScript & Languages
- **typescript**: Generics, discriminated unions, type architectures
- **type-safety**: Fix errors, enforce strict checking
- **python**: Idiomatic Python with type hints

### API & Backend
- **api-design**: REST/GraphQL API design patterns
- **backend-api**: Node.js APIs with middleware patterns
- **graphql**: Schema, resolvers, subscriptions
- **document-api**: API documentation from source code
- **error-handling**: Error types and recovery strategies

### Data & Databases
- **data-model**: Schema design, normalization, indexing
- **data-validation**: Validate data at every boundary
- **database-integration**: Prisma, Supabase, MongoDB clients
- **database-migration**: Safe schema changes, zero downtime
- **sql**: Query design and optimization

### Integration & Feature Services
- **auth**: OAuth2, JWT, RBAC, SSO authentication
- **payments**: Stripe integration, subscriptions, webhooks
- **email**: Transactional email templates and infrastructure
- **file-upload**: Secure uploads with progress and storage
- **search**: Full-text search, autocomplete, filters
- **real-time**: WebSockets, SSE, live collaboration
- **caching**: Multi-layer caching and invalidation
- **ai-integration**: LLMs, RAG, streaming, prompt engineering
- **firebase**: Firestore, Auth, Functions, Hosting
- **webhooks-events**: Webhooks, message queues, async workflows
- **internationalization**: RTL, translation, locale support

### Infrastructure & DevOps
- **docker**: Optimized containers and compose configurations
- **ci-pipeline**: CI/CD pipelines and build optimization
- **deployment**: Zero-downtime deployments with rollback
- **devops-iac**: Terraform, Pulumi, infrastructure provisioning
- **environments**: Dev/staging/prod parity, feature flags, secrets
- **monitoring**: Logging, error tracking, APM, alerting

### Version Control & Release
- **git-commit**: Atomic commits with conventional messages
- **branching-strategy**: Git branching models for teams
- **pr-workflow**: PR creation, review, merge automation
- **code-review**: Bugs, security, style, tests review
- **changelog**: Structured changelogs from git history
- **release-management**: Versioning, tags, coordinated deployment

### Quality & Performance
- **testing-strategy**: Test pyramid, coverage, strategy
- **write-tests**: Thorough tests following conventions
- **performance-optimize**: Profile and optimize bottlenecks
- **debug**: Reproduce, isolate, fix, verify
- **troubleshoot**: Read errors, isolate root cause, apply fixes
- **security-audit**: Secrets, injection, auth vulnerabilities
- **refactor**: Identify code smells and refactor safely
- **migrate-deps**: Safely upgrade or replace dependencies

### Project Setup
- **project-init**: Scaffold projects with proper structure
- **config-setup**: Linters, formatters, type checkers config
- **onboarding**: Project docs and contributor guides
- **plan-feature**: Break features into implementation steps

### SEO
- **seo**: Next.js metadata, OG, crawlability

---

## Installation

These skills are installed in OpenCode's `.opencode/skills/` directory. They're auto-discovered. No configuration needed.

```bash
# Clone the repo
git clone https://github.com/jucish2019-a11y/Supercharge_opencode.git

# Copy skills into your project's .opencode/skills/ directory
cp -r Supercharge_opencode/.opencode/skills/* /your/project/.opencode/skills/
```

Or install individual skills by copying a skill's folder into your project's `.opencode/skills/` directory:

```bash
# Example: install just the design-foundations skill
cp -r Supercharge_opencode/.opencode/skills/design-foundations /your/project/.opencode/skills/
```

Each skill is a single `SKILL.md` file: self-contained, no dependencies.

---

## Skill Format

Every skill follows the same structure:

```markdown
---
name: skill-name
description: One-line description of what the skill does
---

## What I do
Brief summary of the skill's capabilities.

## When to use me
Specific scenarios where the skill applies.

## How I work
### Checker mode (auditing existing work)
Step-by-step audit process...

### Applier mode (building from scratch)
Step-by-step creation process...

## [Skill-specific sections]
Theory, implementation details, code examples, checklists...

## Quality checklist
- [ ] Item 1
- [ ] Item 2

## Anti-patterns I avoid
- Bad pattern 1: why it's bad
- Bad pattern 2: why it's bad
```

---

## Philosophy

> "To truly be adept at designing something, you have to understand how it works. Otherwise, you aren't designing. You're creating a veneer. You're drawing ponies." (David Kadavy)

Every skill in this library teaches the **principle behind the recommendation**. Not just "use a type scale" but *why*: because dimensions that relate through intentional ratios create harmony that the eye registers even when the mind can't name it. Not just "don't use cyan-on-dark" but *why*: because that sameness has become a signal that the output was generated, not designed.

The goal isn't more skills. It's better decisions. Skills are how you get there.

---

## License

MIT

---

*Inspired by [design-for-ai](https://ryanthedev.github.io/design-for-ai/) · Principles from Design for Hackers by David Kadavy*