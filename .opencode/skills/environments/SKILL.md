---
name: environments
description: Manage dev, staging, and production environments — feature flags, environment variables, secrets management, and environment parity
---

## What I do

I set up and manage environments across the development lifecycle:

- **Environment strategy** — Dev, staging, production — what goes where, how they differ
- **Environment variables** — .env conventions, validation, TypeScript-typed env
- **Secrets management** — Where to store secrets, rotation, access control
- **Feature flags** — LaunchDarkly, custom flags, gradual rollout, cleanup
- **Environment parity** — Making dev/prod differences explicit and minimal
- **Preview environments** — Per-PR deploy previews for testing

## When to use me

Use this skill when:
- Setting up environment configuration for a new project
- Managing secrets and environment variables
- Implementing feature flags for gradual rollouts
- Setting up staging environments or preview deployments
- Debugging environment-specific issues
- Planning environment promotion (dev → staging → production)

## Environment strategy

### Three-environment model (recommended)

```
Development (local):
  Purpose: Write and test code locally
  Data: Seed scripts, sample data, no real users
  Config: .env.local, local database, mock external services
  Cost: Free (developer machine)

Staging (pre-production):
  Purpose: Validate changes before production, QA testing, demos
  Data: Production-like (sanitized copy or generated)
  Config: .env.staging, staging database, real external service test keys
  Cost: Same infrastructure as production, smaller scale
  
Production:
  Purpose: Serve real users
  Data: Real user data
  Config: .env.production, production database, real service keys
  Cost: Full scale
```

### Preview environments (per-PR)

```
For every pull request:
  - Deploy the branch to a preview URL (e.g., pr-123.staging.app.com)
  - Use staging infrastructure but branch-specific code
  - Auto-destroy when PR is merged or closed
  - Share the URL in the PR for reviewers

Tools: Vercel Preview Deployments, Netlify Deploy Previews, custom with Kubernetes
```

## Environment variables

### .env file conventions

```
.env                  ← Default values (checked into git, no secrets)
.env.local            ← Local overrides (gitignored, developer-specific)
.env.development      ← Development overrides (checked in, no secrets)
.env.staging          ← Staging overrides (checked in, no secrets)
.env.production       ← Production reference (checked in, NO secrets — only key names)

# NEVER commit files containing real secrets:
.env.local            ← gitignored
.env.*.local          ← gitignored
.production.secrets   ← gitignored
```

### .env file content

```bash
# .env (checked in — default/reference values)
APP_NAME=MyApp
APP_URL=http://localhost:3000
DATABASE_URL=postgresql://localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379
LOG_LEVEL=debug

# Keys that NEED secrets (reference only — actual values in .env.local)
# GOOGLE_CLIENT_ID=
# GOOGLE_CLIENT_SECRET=
# STRIPE_SECRET_KEY=
# STRIPE_WEBHOOK_SECRET=
# SENTRY_DSN=
```

### Typed environment variables

```ts
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']).default('development'),
  APP_URL: z.string().url(),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  GOOGLE_CLIENT_ID: z.string().min(1),
  GOOGLE_CLIENT_SECRET: z.string().min(1),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),
  SENTRY_DSN: z.string().url().optional(),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

function validateEnv() {
  const parsed = envSchema.safeParse(process.env);
  if (!parsed.success) {
    console.error('Invalid environment variables:', parsed.error.flatten().fieldErrors);
    throw new Error('Invalid environment variables');
  }
  return parsed.data;
}

export const env = validateEnv();

// Usage — fully typed!
const db = new PrismaClient({ datasourceUrl: env.DATABASE_URL });
```

### Next.js environment variables

```
Rules:
  NEXT_PUBLIC_* → exposed to browser (use for public keys, API URLs)
  No prefix     → server-only (secrets, database URLs, API keys)

# .env.local
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

DATABASE_URL=postgresql://...          # Server only
STRIPE_SECRET_KEY=sk_test_...         # Server only
GOOGLE_CLIENT_SECRET=...              # Server only
```

Never put secrets in `NEXT_PUBLIC_*` variables — they're embedded in the client bundle.

## Secrets management

### Development
```
Method: .env.local files (gitignored)
Storage: Developer's machine only
Rotation: Manual (when compromised)
Access: Developer only
```

### Staging
```
Method: Platform secrets (Vercel env, Doppler, AWS Secrets Manager)
Storage: Encrypted at rest, injected at build/runtime
Rotation: Shared service account keys, rotated quarterly
Access: CI/CD pipeline + team leads
```

### Production
```
Method: Secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler)
Storage: Encrypted, audited access, automatic rotation
Rotation: Automatic for supported services (DB credentials, API keys)
Access: CI/CD pipeline + on-call engineers
Audit: All access logged
```

### Secrets rules
- Never commit secrets to git (even in private repos)
- Never log secrets (redact in logs)
- Never expose secrets to the browser (no NEXT_PUBLIC_ prefix on secrets)
- Never share production secrets with staging (separate keys per environment)
- Rotate secrets on a schedule and on team member departure
- Use least-privilege: different API keys for different services, not one key for everything

## Feature flags

### Simple flags (no library needed)

```ts
// lib/features.ts
const features = {
  newDashboard: process.env.NEXT_PUBLIC_FEATURE_NEW_DASHBOARD === 'true',
  exportCsv: process.env.NEXT_PUBLIC_FEATURE_EXPORT_CSV === 'true',
  darkMode: true, // Always enabled
} as const;

export type Feature = keyof typeof features;

export function isFeatureEnabled(feature: Feature): boolean {
  return features[feature] ?? false;
}

// Usage
if (isFeatureEnabled('newDashboard')) {
  return <NewDashboard />;
}
return <LegacyDashboard />;
```

### Gradual rollout (percentage-based)

```ts
function isRolloutEnabled(feature: string, userId: string, percentage: number): boolean {
  const hash = cyrb53(`${feature}:${userId}`) % 100;
  return hash < percentage;
}

// 20% of users see the new feature
if (isRolloutEnabled('newDashboard', user.id, 20)) {
  return <NewDashboard />;
}
```

### Feature flag lifecycle

```
1. Create: Add flag, default OFF in all environments
2. Test: Enable in development and staging
3. Rollout: Enable for percentage of users in production (10% → 50% → 100%)
4. Stabilize: Keep at 100% for 1-2 weeks, monitor for issues
5. Cleanup: Remove the flag and the old code path
6. Delete: Remove the flag from the configuration

NEVER skip step 5 — dead feature flags accumulate and make the codebase confusing
```

### Cleanup pattern

```ts
// Before cleanup: two code paths
if (isFeatureEnabled('newDashboard')) {
  return <NewDashboard />;
}
return <LegacyDashboard />;

// After cleanup: remove the flag and the old path
return <NewDashboard />;
// Delete: the feature flag from features.ts
// Delete: the LegacyDashboard component
// Delete: the isFeatureEnabled checks
```

## Environment parity

### What should be identical between staging and production

```
Same:
  - Application code (same version deployed)
  - Infrastructure config (same services, same topology)
  - Environment variable names (can differ in values)
  - Database schema (same migrations applied)
  - External service integrations (same providers, test keys)
  - Error handling and logging
```

### What can differ

```
Different:
  - Secret values (separate API keys per environment)
  - Scale (fewer instances, smaller databases)
  - Data (sanitized or synthetic, not real user data)
  - Email delivery (use Mailtrap/Mailhog, not real users)
  - Payment processing (use Stripe test mode, not real charges)
  - Monitoring sensitivity (lower alert thresholds in staging)
```

### What should NOT differ

```
NEVER differ:
  - Code behavior (no if (isStaging) shortcuts)
  - Database schema (same migrations must work in both)
  - API contracts (same request/response shapes)
  - Error handling (don't swallow errors in staging that surface in prod)
  - Feature logic (same feature flag logic in both)
```

## Quality checklist

- [ ] .env.local is in .gitignore
- [ ] No secrets in committed .env files
- [ ] No secrets in NEXT_PUBLIC_ variables
- [ ] Environment variables validated at startup (Zod schema)
- [ ] .env reference file documents all required variables
- [ ] Staging uses separate secrets from production
- [ ] Feature flags have a cleanup lifecycle
- [ ] Preview environments deploy per-PR
- [ ] No code behavior differences between environments
- [ ] Secrets are rotated on schedule
- [ ] Production secrets stored in a secrets manager
- [ ] Departed team members' access revoked immediately

## Anti-patterns I avoid

- Committing .env files with real secrets
- Using NEXT_PUBLIC_ for secret values
- Sharing production API keys across environments
- Using if (process.env.NODE_ENV === 'production') for feature logic — use feature flags
- Skipping env validation — silently failing with undefined is worse than crashing on startup
- Hardcoding configuration values that should be environment-specific
- Dead feature flags that no one cleans up
- Different code behavior between staging and production