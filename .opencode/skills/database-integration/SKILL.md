---
name: database-integration
description: Integrate databases into applications — Prisma with PostgreSQL, Supabase, MongoDB, and choosing the right database for the job with client setup, migrations, queries, and connection management
---

## What I do

I integrate databases into applications with proper client setup, schema management, and query patterns:

- **Database selection** — PostgreSQL, MySQL, MongoDB, Supabase, PlanetScale — when to use each
- **Prisma ORM** — Schema definition, migrations, client generation, query patterns
- **Supabase** — PostgreSQL + real-time + auth + storage — full backend as a service
- **MongoDB** — Document database patterns, Mongoose ODM, aggregation pipelines
- **Connection management** — Pooling, serverless connections, edge compatibility
- **Query optimization** — Indexing, N+1 prevention, selective field loading

## When to use me

Use this skill when:
- Choosing a database for a new project
- Setting up Prisma with PostgreSQL
- Integrating Supabase for rapid backend development
- Working with MongoDB for document-oriented data
- Managing database connections in serverless environments
- Optimizing database queries and schema design

## Database selection guide

| Database | Best for | Trade-offs |
|----------|----------|------------|
| **PostgreSQL** (via Prisma) | Relational data, complex queries, ACID transactions, most apps | Requires migration management, more setup |
| **Supabase** (PostgreSQL + BaaS) | Rapid development, real-time, auth + storage built-in | Vendor lock-in, less control over infra |
| **MongoDB** | Document data, flexible schema, high write throughput, logs/events | No joins, eventual consistency options, large document sizes |
| **PlanetScale** (Serverless MySQL) | Serverless/edge deployments, branch-based schema changes | MySQL limitations, paid for production |
| **SQLite** (via Turso) | Edge/local-first, embedded, simple deployment | Limited concurrency, no network access |
| **Firebase Firestore** | Real-time, mobile-first, serverless (see firebase skill) | No complex queries, denormalization required |

### Decision tree

```
Need real-time subscriptions?         → Supabase or Firestore
Complex relational data?              → PostgreSQL + Prisma
Flexible/changing schema?             → MongoDB
Edge/serverless deployment?           → PlanetScale or Turso
Rapid MVP, want auth+storage+DB?     → Supabase
Document storage, event logging?     → MongoDB
Maximum control, traditional app?    → PostgreSQL + Prisma
```

## Prisma + PostgreSQL

### Setup

```bash
npm install prisma @prisma/client
npx prisma init
```

### Schema definition

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  avatarUrl     String?
  role          Role      @default(MEMBER)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  projects      ProjectMember[]
  tasks         Task[]
}

model Project {
  id            String    @id @default(cuid())
  name          String
  description   String?
  status        ProjectStatus @default(ACTIVE)
  isPublic      Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  members       ProjectMember[]
  tasks         Task[]

  @@index([status])
  @@index([createdAt])
}

model ProjectMember {
  id        String   @id @default(cuid())
  role      MemberRole @default(MEMBER)
  joinedAt  DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    String
  project   Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  projectId String

  @@unique([userId, projectId])
  @@index([projectId])
  @@index([userId])
}

model Task {
  id          String     @id @default(cuid())
  title       String
  description String?
  status      TaskStatus @default(TODO)
  priority    Priority   @default(MEDIUM)
  dueDate     DateTime?
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  project     Project    @relation(fields: [projectId], references: [id], onDelete: Cascade)
  projectId   String
  assignee    User?      @relation(fields: [assigneeId], references: [id], onDelete: SetNull)
  assigneeId  String?

  @@index([projectId, status])
  @@index([assigneeId])
  @@index([dueDate])
}

enum Role { OWNER ADMIN MEMBER }
enum MemberRole { OWNER ADMIN MEMBER VIEWER }
enum ProjectStatus { ACTIVE ARCHIVED DELETED }
enum TaskStatus { TODO IN_PROGRESS IN_REVIEW DONE }
enum Priority { LOW MEDIUM HIGH URGENT }
```

### Database client

```ts
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const db =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = db;
```

### Migration workflow

```bash
# Create a migration from schema changes
npx prisma migrate dev --name add-tasks

# Apply migrations in production
npx prisma migrate deploy

# Generate Prisma Client after schema changes
npx prisma generate

# Reset database (development only!)
npx prisma migrate reset

# Seed the database
npx prisma db seed
```

### Seed script

```ts
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
const db = new PrismaClient();

async function main() {
  const user = await db.user.create({
    data: {
      email: 'demo@example.com',
      name: 'Demo User',
      role: 'OWNER',
    },
  });

  const project = await db.project.create({
    data: {
      name: 'My First Project',
      description: 'A demo project to get started',
      members: {
        create: { userId: user.id, role: 'OWNER' },
      },
      tasks: {
        create: [
          { title: 'Complete onboarding', status: 'TODO', assigneeId: user.id },
          { title: 'Invite team members', status: 'TODO' },
        ],
      },
    },
  });

  console.log({ user, project });
}

main()
  .catch(console.error)
  .finally(() => db.$disconnect());
```

### Query patterns

```ts
// Create with related data
const project = await db.project.create({
  data: {
    name: 'New Project',
    members: {
      create: { userId: user.id, role: 'OWNER' },
    },
  },
  include: { members: true },
});

// Read with relations
const project = await db.project.findUnique({
  where: { id: projectId },
  include: {
    members: { include: { user: { select: { id: true, name: true, email: true } } } },
    tasks: {
      where: { status: 'TODO' },
      orderBy: { priority: 'desc' },
      take: 20,
    },
  },
});

// Select specific fields (performance)
const projects = await db.project.findMany({
  where: { status: 'ACTIVE' },
  select: {
    id: true,
    name: true,
    status: true,
    _count: { select: { tasks: true, members: true } },
  },
  orderBy: { updatedAt: 'desc' },
});

// Update with relation
await db.project.update({
  where: { id: projectId },
  data: {
    name: 'Updated Name',
    tasks: {
      update: {
        where: { id: taskId },
        data: { status: 'DONE' },
      },
    },
  },
});

// Transaction
await db.$transaction(async (tx) => {
  await tx.task.update({
    where: { id: taskId },
    data: { assigneeId: newAssigneeId },
  });
  await tx.notification.create({
    data: {
      userId: newAssigneeId,
      message: 'You have been assigned a task',
    },
  });
});

// Aggregation
const taskCounts = await db.task.groupBy({
  by: ['status'],
  where: { projectId },
  _count: { status: true },
});
```

### Connection management for serverless

```ts
// For Vercel / serverless — connection pooling
// Use Prisma with connection pooling via pgbouncer or Prisma Accelerate
//
// DATABASE_URL="postgresql://user:pass@host:6543/db?pgbouncer=true"
// DIRECT_URL="postgresql://user:pass@host:5432/db"
//
// schema.prisma:
// datasource db {
//   provider  = "postgresql"
//   url       = env("DATABASE_URL")
//   directUrl = env("DIRECT_URL")
// }

// Prisma Accelerate (recommended for serverless)
// npx prisma generate --accelerate
// const db = new PrismaClient().$extends(withAccelerate())
```

## Supabase integration

### Setup

```bash
npm install @supabase/supabase-js
```

### Client

```ts
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

// Server-side client (with service role — bypasses RLS)
export const supabaseAdmin = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
```

### Auth with Supabase

```tsx
import { supabase } from '@/lib/supabase';

// Sign up
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'securepassword',
});

// Sign in with OAuth
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: { redirectTo: `${window.location.origin}/auth/callback` },
});

// Listen to auth state
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') setUser(session?.user);
  if (event === 'SIGNED_OUT') setUser(null);
});
```

### Real-time subscriptions

```ts
// Subscribe to inserts on the tasks table
const channel = supabase
  .channel('tasks-changes')
  .on('postgres_changes', {
    event: 'INSERT',
    schema: 'public',
    table: 'tasks',
    filter: `project_id=eq.${projectId}`,
  }, (payload) => {
    setTasks(prev => [...prev, payload.new]);
  })
  .subscribe();

// Cleanup
useEffect(() => {
  return () => { supabase.removeChannel(channel); };
}, []);
```

### Row Level Security (RLS)

```sql
-- Enable RLS
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Users can only see projects they're members of
CREATE POLICY "Users can view their projects"
  ON projects FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM project_members
      WHERE project_members.project_id = projects.id
      AND project_members.user_id = auth.uid()
    )
  );

-- Only project admins can update
CREATE POLICY "Admins can update projects"
  ON projects FOR UPDATE
  USING (
    EXISTS (
      SELECT 1 FROM project_members
      WHERE project_members.project_id = projects.id
      AND project_members.user_id = auth.uid()
      AND project_members.role IN ('owner', 'admin')
    )
  );

-- Anyone authenticated can create projects
CREATE POLICY "Authenticated users can create projects"
  ON projects FOR INSERT
  WITH CHECK (auth.uid() IS NOT NULL);
```

## MongoDB integration

### Setup with Mongoose

```bash
npm install mongoose
```

### Connection

```ts
// lib/mongodb.ts
import mongoose from 'mongoose';

const MONGODB_URI = process.env.MONGODB_URI!;

let cached = global.mongoose;

if (!cached) {
  cached = global.mongoose = { conn: null, promise: null };
}

export async function connectMongoDB() {
  if (cached.conn) return cached.conn;

  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI, {
      bufferCommands: false,
    });
  }

  cached.conn = await cached.promise;
  return cached.conn;
}
```

### Schema definition

```ts
// models/Project.ts
import { Schema, model } from 'mongoose';

const projectSchema = new Schema({
  name: { type: String, required: true, trim: true, maxlength: 100 },
  description: { type: String, maxlength: 1000 },
  status: { type: String, enum: ['active', 'archived', 'deleted'], default: 'active' },
  createdBy: { type: Schema.Types.ObjectId, ref: 'User', required: true },
  members: [{
    userId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
    role: { type: String, enum: ['owner', 'admin', 'member', 'viewer'], default: 'member' },
    joinedAt: { type: Date, default: Date.now },
  }],
  tags: [{ type: String }],
  taskCount: { type: Number, default: 0 },
}, {
  timestamps: true,
});

projectSchema.index({ status: 1, updatedAt: -1 });
projectSchema.index({ 'members.userId': 1 });

export const Project = model('Project', projectSchema);
```

### MongoDB query patterns

```ts
// Create
const project = await Project.create({
  name: 'New Project',
  createdBy: userId,
  members: [{ userId, role: 'owner' }],
});

// Read with population
const project = await Project.findById(projectId)
  .populate('createdBy', 'name email')
  .populate('members.userId', 'name email')
  .lean();

// Update with atomic operators
await Project.findByIdAndUpdate(projectId, {
  $set: { name: 'Updated Name' },
  $inc: { taskCount: 1 },
  $push: { tags: 'important' },
});

// Aggregation pipeline
const taskStats = await Task.aggregate([
  { $match: { projectId: new ObjectId(projectId) } },
  { $group: {
    _id: '$status',
    count: { $sum: 1 },
    avgPriority: { $avg: { $switch: {
      branches: [
        { case: { $eq: ['$priority', 'urgent'] }, then: 4 },
        { case: { $eq: ['$priority', 'high'] }, then: 3 },
        { case: { $eq: ['$priority', 'medium'] }, then: 2 },
        { case: { $eq: ['$priority', 'low'] }, then: 1 },
      ],
      default: 0,
    }}},
  }},
]);
```

## Quality checklist

- [ ] Database selected based on data model and access patterns
- [ ] Prisma schema follows naming conventions and has indexes
- [ ] Migrations are committed to version control
- [ ] Seed script exists for development data
- [ ] Connection pooling for serverless (pgbouncer or Prisma Accelerate)
- [ ] Prisma client singleton pattern (no hot-reload connection leaks)
- [ ] Queries use `select` and `include` to avoid over-fetching
- [ ] Indexes on frequently queried fields
- [ ] Foreign keys with proper onDelete behavior
- [ ] RLS or security rules for data access control
- [ ] Real-time subscriptions cleaned up on unmount
- [ ] Transactions for multi-step operations
- [ ] Database URL from environment variable (not hardcoded)

## Anti-patterns I avoid

- Choosing a database before understanding the data model
- Using Prisma without indexes — queries degrade as data grows
- Not using connection pooling in serverless — connection exhaustion
- Creating multiple PrismaClient instances — connection leak
- Over-fetching with `include` when `select` suffices
- N+1 queries — use includes or batch queries
- MongoDB for highly relational data — use PostgreSQL
- PostgreSQL for schemaless event data — use MongoDB
- Not committing migrations — schema drift between environments
- Hardcoding database URLs — use environment variables