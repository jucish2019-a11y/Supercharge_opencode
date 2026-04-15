---
name: docker
description: Create and optimize Dockerfiles, compose files, and container configurations for development and production
---

## What I do

I build containerized applications using Docker best practices:

- **Dockerfiles** — Multi-stage builds, minimal images, layer caching optimization
- **Docker Compose** — Development environments, service orchestration, networking
- **Optimization** — Image size reduction, build speed, security hardening
- **Configuration** — Environment variables, volumes, health checks, logging

## When to use me

Use this skill when:
- Writing or optimizing a Dockerfile
- Setting up Docker Compose for development or deployment
- Debugging container build or runtime issues
- Reducing Docker image size
- Configuring container networking or volumes

## How I work

1. **Discover the existing setup** — Check for existing Dockerfiles, compose files, `.dockerignore`, and the project's language/runtime requirements.
2. **Choose the right base image** — Prefer official images, use `alpine` or `slim` variants where appropriate, pin version tags (never `latest` in production).
3. **Optimize build layers**:
   - Order instructions from least to most frequently changing
   - Combine related `RUN` commands to reduce layers
   - Use `COPY --from=builder` for multi-stage builds
   - Leverage build cache by copying dependency files before source code
4. **Security hardening**:
   - Run as non-root user (`USER` directive)
   - No secrets in images — use build args or runtime env vars
   - Minimize installed packages
   - Use `READONLY` root filesystem where possible
5. **Compose configuration** — Define services with proper health checks, restart policies, and dependency ordering.
6. **Test the result** — Build, run, verify the container works, check image size.

## Dockerfile template (multi-stage)

```dockerfile
FROM language:version AS builder
WORKDIR /app
COPY dependency-files ./
RUN install dependencies
COPY . .
RUN build command

FROM language:version-slim AS runtime
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=builder /app/output ./
USER appuser
EXPOSE 8080
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
CMD ["start-command"]
```

## Key principles

- Pin all image versions (`node:20.11.0` not `node:latest`)
- Use `.dockerignore` to exclude node_modules, `.git`, `__pycache__`, etc.
- One process per container
- Environment-specific config via environment variables, not baked into the image
- Use `HEALTHCHECK` for orchestration-aware containers
- Prefer multi-stage builds to keep images minimal
- Never store secrets in Dockerfiles or compose files

## Anti-patterns I avoid

- Running as root inside the container
- Installing unnecessary packages (vim, curl, etc.) in production images
- Binding to `0.0.0.0` without understanding the security implications
- Using `ADD` instead of `COPY` for local files
- Not using `.dockerignore`
- Large single-stage builds for production images