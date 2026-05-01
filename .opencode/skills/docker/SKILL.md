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

## Multi-stage builds

### Node.js

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:20-alpine AS production
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy only necessary files
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .

USER nodejs
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1

CMD ["node", "server.js"]
```

### Python

```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app

# Copy only installed packages
COPY --from=builder /root/.local /root/.local
COPY . .

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

EXPOSE 8000
HEALTHCHECK CMD curl -f http://localhost:8000/health || exit 1
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0"]
```

## Docker Compose

### Development

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - '3000:3000'
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    ports:
      - '5432:5432'
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'

volumes:
  postgres_data:
```

### Production

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '3000:3000'
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${DB_USER} -d ${DB_NAME}']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    restart: unless-stopped

volumes:
  postgres_data:
```

## Security hardening

```dockerfile
# Run as non-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Don't run as root
FROM node:20-alpine
RUN adduser -S appuser
USER appuser

# Read-only root filesystem
 docker run --read-only --tmpfs /tmp myimage

# Drop capabilities
 docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage

# No new privileges
 docker run --security-opt=no-new-privileges:true myimage
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