---
name: deployment
description: Plan and execute deployments with rollback strategies, health checks, and zero-downtime delivery
---

## What I do

I handle deployment planning and execution:

- **Deployment strategies** — Blue-green, canary, rolling, feature-flagged releases
- **Environment management** — Dev, staging, production parity and configuration
- **Health checks** — Readiness and liveness probes, smoke tests after deploy
- **Rollback** — Safe rollback procedures, data migration rollbacks
- **CI/CD pipelines** — Build, test, deploy automation

## When to use me

Use this skill when:
- Deploying a new service or application
- Setting up a CI/CD pipeline
- Planning a deployment strategy for a critical release
- Creating rollback plans
- Debugging deployment failures

## How I work

1. **Understand the target** — What platform (AWS, GCP, Azure, Vercel, Fly.io, bare metal)? What's the current deployment method? What are the existing environments?
2. **Assess risk level** — How critical is the service? What's the impact of downtime? Does the change include database migrations? This determines the deployment strategy.
3. **Choose a strategy**:
   - Low risk / simple app → Rolling update
   - High risk / needs instant rollback → Blue-green
   - Gradual validation needed → Canary
   - Feature under development → Feature flag
4. **Define the pipeline**:
   - Build → Test → Lint → Type check (fast feedback, fail early)
   - Integration tests → Staging deploy → Smoke tests
   - Production deploy → Health check → Smoke test → Monitor
5. **Plan rollback** — What triggers a rollback? How do you execute it? What about data migrations that can't be auto-reversed?
6. **Set up monitoring** — Log aggregation, error tracking, deployment annotations in metrics dashboards.
7. **Document the runbook** — Step-by-step deploy, verify, and rollback procedures.

## Platform-specific deployment

### Vercel

```bash
# Production deployment
vercel --prod

# Preview deployment (per PR)
vercel

# Environment variables
vercel env add DATABASE_URL production
```

### AWS (ECS/Fargate)

```yaml
# docker-compose.yml for ECS
deployment:
  strategy: rolling
  maxUnavailable: 0
  maxSurge: 1

healthcheck:
  path: /health
  interval: 30s
  timeout: 5s
  retries: 3
```

### Fly.io

```bash
# Deploy
fly deploy

# Scale
fly scale count 3

# Rollback
fly rollback
```

## Health check endpoint

```ts
// app/health/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    memory: process.memoryUsage().heapUsed < 500 * 1024 * 1024,
  };

  const healthy = Object.values(checks).every(Boolean);

  return NextResponse.json(
    { status: healthy ? 'healthy' : 'unhealthy', checks },
    { status: healthy ? 200 : 503 }
  );
}

async function checkDatabase(): Promise<boolean> {
  try {
    await db.$queryRaw`SELECT 1`;
    return true;
  } catch {
    return false;
  }
}
```

## Database migration during deploy

```
Safe migration strategy:
1. Deploy code that can handle BOTH old and new schema
2. Run migration (backward-compatible: add column, create table)
3. Verify migration succeeded
4. Deploy code that uses new schema
5. Later: run cleanup migration (drop old column)

Never:
- Drop columns before code stops using them
- Rename columns in place (add new, migrate data, drop old)
- Make non-backward-compatible changes in a single deploy
```

## Deployment checklist

Before every deployment:
- [ ] All tests pass in CI
- [ ] Database migrations are backward-compatible (or have a plan)
- [ ] Environment variables are set on the target
- [ ] Health check endpoint is configured
- [ ] Rollback procedure is documented and tested
- [ ] Monitoring/alerting is in place
- [ ] Secrets are not in the codebase

After deployment:
- [ ] Health check returns green
- [ ] Smoke tests pass on production
- [ ] Error rate is within baseline
- [ ] Key user flows work manually
- [ ] Deployment is tagged in git

## Key principles

- Small, frequent deployments are safer than large, rare ones
- Every deployment must be reversible within minutes
- Database migrations must be backward-compatible (deploy in two phases: add column, then use column)
- Never deploy on Friday unless you have weekend coverage
- Automate everything that can be automated — manual steps are error-prone
- Feature flags let you deploy without releasing

## Anti-patterns I avoid

- Deploying database schema and code in one step
- SSH-ing into servers to deploy manually
- No health checks after deploy
- Deleting old deployment before verifying new one works
- Skipping staging environment for production deploys
- Hardcoded configuration values instead of environment variables