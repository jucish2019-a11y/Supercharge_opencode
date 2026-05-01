---
name: ci-pipeline
description: Set up and optimize CI/CD pipelines, fix build failures, and improve build performance
---

## What I do

I set up and optimize CI/CD pipelines:

- **GitHub Actions** — Workflow design, caching, matrix builds
- **Build optimization** — Parallel jobs, artifact management, dependency caching
- **Quality gates** — Lint, type check, test, security scan
- **Deployment automation** — Staging and production pipelines
- **Monorepo CI** — Turborepo, Nx, changeset-based workflows

## When to use me

Use this skill when:
- Setting up CI/CD for a new project
- Optimizing slow build times
- Debugging failing CI pipelines
- Adding quality gates to the build process
- Setting up monorepo CI workflows

## GitHub Actions templates

### Node.js project

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run typecheck

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:ci

  build:
    runs-on: ubuntu-latest
    needs: [lint, typecheck, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist
```

### Matrix builds

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
        database: [postgres, mysql]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
        env:
          DATABASE_URL: ${{ matrix.database == 'postgres' && 'postgresql://...' || 'mysql://...' }}
```

## Caching strategies

### npm cache

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
```

### Turborepo remote caching

```yaml
- name: Setup Turborepo cache
  uses: dtinth/setup-github-actions-caching-for-turbo@v1

- name: Build
  run: npx turbo run build --cache-dir=.turbo
```

### Custom cache

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      node_modules
      .next/cache
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

## Monorepo CI

```yaml
name: Monorepo CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      apps: ${{ steps.changes.outputs.apps }}
      packages: ${{ steps.changes.outputs.packages }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: changes
        with:
          filters: |
            apps:
              - 'apps/**'
            packages:
              - 'packages/**'

  build-apps:
    needs: changes
    if: ${{ needs.changes.outputs.apps == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx turbo run build --filter=./apps/*
```

## Quality gates

```yaml
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci

      # Lint
      - run: npm run lint

      # Type check
      - run: npm run typecheck

      # Test with coverage
      - run: npm run test:coverage

      # Security audit
      - run: npm audit --audit-level=moderate

      # Build
      - run: npm run build

      # Bundle size check
      - run: npm run analyze
```

## Deployment pipeline

```yaml
jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: [lint, typecheck, test, build]
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist
      - run: npm run deploy:staging

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist
      - run: npm run deploy:production
```

## Key principles

- Fail fast: lint and type check before tests
- Parallelize independent jobs
- Cache dependencies aggressively
- Use artifacts to pass build outputs between jobs
- Pin action versions (not `@master`)
- Run CI on every PR and push to main
- Require CI pass before merge

## Anti-patterns I avoid

- Running everything sequentially in one job
- Not caching dependencies
- Using `latest` for action versions
- Skipping CI for "trivial" changes
- Not pinning Node.js/runtime versions
- Running tests without lint/type check first
- Not using artifacts — rebuilding in every job
- Allowing merges with failing CI