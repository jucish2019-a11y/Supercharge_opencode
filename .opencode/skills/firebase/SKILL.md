---
name: firebase
description: Integrate Firebase services — Firestore, Authentication, Storage, Cloud Functions, Hosting, and Emulator Suite for rapid full-stack development
---

## What I do

I integrate Firebase services into web applications for rapid full-stack development:

- **Firestore** — NoSQL database: data modeling, queries, real-time listeners, security rules
- **Authentication** — Firebase Auth: email/password, OAuth, phone, custom tokens
- **Storage** — Cloud Storage: file uploads, signed URLs, security rules
- **Cloud Functions** — Serverless functions: triggers, scheduled, HTTP-callable
- **Hosting** — Firebase Hosting: static sites, SSR rewrites, CDN
- **Emulator Suite** — Local development with all services emulated

## When to use me

Use this skill when:
- Building a new app with Firebase as the backend
- Modeling data in Firestore (different from SQL — requires denormalization)
- Writing Firestore security rules
- Implementing Firebase Auth with social providers
- Setting up file uploads with Cloud Storage
- Writing Cloud Functions for backend logic
- Configuring the Firebase Emulator Suite for local development

## When to use Firebase vs alternatives

| Use Firebase when | Use alternatives when |
|---|---|
| Rapid prototyping / MVP | Complex relational data |
| Real-time features (chat, collaboration) | Complex transactions / multi-entity writes |
| Simple auth needs (social + email) | Enterprise SSO / SAML |
| Mobile + web from same backend | Predictable cost at scale |
| Serverless backend preferred | Dedicated backend with long-running processes |
| Small team, fast iteration | Large team, strict schema control |

## Project setup

### Installation

```bash
npm install firebase
npm install -D firebase-tools
```

### Client configuration

```ts
// lib/firebase.ts
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';
import { getStorage } from 'firebase/storage';
import { getFunctions } from 'firebase/functions';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);

export const db = getFirestore(app);
export const auth = getAuth(app);
export const storage = getStorage(app);
export const functions = getFunctions(app);
```

### Emulator configuration (local development)

```ts
// lib/firebase.ts — connect to emulators in development
if (process.env.NEXT_PUBLIC_USE_EMULATORS === 'true') {
  connectFirestoreEmulator(db, 'localhost', 8080);
  connectAuthEmulator(auth, 'http://localhost:9099');
  connectStorageEmulator(storage, 'localhost', 9199);
  connectFunctionsEmulator(functions, 'localhost', 5001);
}
```

```yaml
# firebase.json
{
  "emulators": {
    "auth": { "port": 9099 },
    "firestore": { "port": 8080 },
    "storage": { "port": 9199 },
    "functions": { "port": 5001 },
    "ui": { "enabled": true, "port": 4000 }
  }
}
```

## Firestore data modeling

### Key principle: Firestore is NOT SQL

```
SQL mindset:  Normalize data, avoid duplication, JOIN at query time
Firestore:    Denormalize for read performance, duplicate data, no JOINs

Rules:
1. Optimize for the reads, not the writes (reads are 10x more frequent)
2. Duplicate data to avoid queries that Firestore can't do
3. Keep related data together in the same document or subcollection
4. Use subcollections for one-to-many relationships
5. Use flat documents with embedded arrays for small, bounded lists
6. Use references for large, unbounded lists
7. Denormalize frequently-read fields into parent documents
```

### Document structure

```
projects/
  {projectId}/
    name: string
    description: string
    status: 'active' | 'archived'
    createdBy: uid
    createdAt: timestamp
    updatedAt: timestamp
    memberCount: number          ← denormalized counter
    tags: string[]               ← embedded array (small, bounded)

    members/                     ← subcollection
      {uid}/
        role: 'owner' | 'admin' | 'member'
        joinedAt: timestamp

    tasks/                       ← subcollection
      {taskId}/
        title: string
        status: 'todo' | 'in_progress' | 'done'
        assigneeId: uid | null
        dueDate: timestamp | null
        createdAt: timestamp
```

### When to use subcollections vs top-level collections

```
Use subcollections when:
  - Data belongs exclusively to the parent (tasks belong to a project)
  - You need to query within the parent's scope (tasks in project X)
  - The parent ID is needed for access control (members of project X)

Use top-level collections when:
  - Data is accessed independently of parent (users, organizations)
  - You need to query across all parents (all tasks across all projects)
  - Data is shared between multiple parents

For cross-project queries (like "all my tasks"):
  Create a denormalized top-level collection OR
  Use a Cloud Function to maintain an index
```

### Counter patterns

```
Denormalized counters (memberCount, taskCount):
  Update atomically with FieldValue.increment:

  await updateDoc(projectRef, {
    memberCount: increment(1),
  });

  // Decrement on removal
  await updateDoc(projectRef, {
    memberCount: increment(-1),
  });
```

### Querying

```ts
import { collection, query, where, orderBy, limit, getDocs } from 'firebase/firestore';

// Simple query
const q = query(
  collection(db, 'projects'),
  where('status', '==', 'active'),
  orderBy('updatedAt', 'desc'),
  limit(20)
);
const snapshot = await getDocs(q);

// Subcollection query
const tasksQuery = query(
  collection(db, 'projects', projectId, 'tasks'),
  where('assigneeId', '==', userId),
  where('status', '==', 'todo'),
  orderBy('dueDate')
);

// Composite queries require a composite index
// Firestore auto-creates indexes for single-field queries
// Composite (2+ fields) requires manual index creation
// Deploy index: firebase deploy --only firestore:indexes
```

### Query limitations

```
Firestore CANNOT do:
  - OR queries across different fields (use separate queries + merge)
  - NOT (!=) queries efficiently (scan + filter in client)
  - Full-text search (use Algolia or Elastic for search)
  - Aggregation across documents (count, sum — use Cloud Functions)
  - JOIN-like queries across collections (denormalize instead)
  - Queries on arrays with multiple conditions (one array-contains per query)
```

### Real-time listeners

```ts
import { onSnapshot, doc, collection, query, where } from 'firebase/firestore';

// Listen to a single document
const unsubscribe = onSnapshot(doc(db, 'projects', projectId), (snapshot) => {
  const project = { id: snapshot.id, ...snapshot.data() };
  setProject(project);
});

// Listen to a query (real-time list)
const unsubscribe = onSnapshot(
  query(collection(db, 'projects'), where('status', '==', 'active')),
  (snapshot) => {
    const projects = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    setProjects(projects);
  }
);

// Cleanup — ALWAYS unsubscribe to avoid memory leaks
useEffect(() => {
  const unsubscribe = onSnapshot(/* ... */);
  return () => unsubscribe();
}, []);
```

### Batch writes

```ts
import { writeBatch, doc } from 'firebase/firestore';

async function createProjectWithOwner(projectData: any, userId: string) {
  const batch = writeBatch(db);

  const projectRef = doc(collection(db, 'projects'));
  const memberRef = doc(db, 'projects', projectRef.id, 'members', userId);

  batch.set(projectRef, {
    ...projectData,
    createdBy: userId,
    memberCount: 1,
    createdAt: serverTimestamp(),
    updatedAt: serverTimestamp(),
  });

  batch.set(memberRef, {
    role: 'owner',
    joinedAt: serverTimestamp(),
  });

  await batch.commit();
}
```

### Transactions

```ts
import { runTransaction } from 'firebase/firestore';

async function addMemberToProject(projectId: string, userId: string, role: string) {
  await runTransaction(db, async (transaction) => {
    const projectRef = doc(db, 'projects', projectId);
    const memberRef = doc(db, 'projects', projectId, 'members', userId);

    const projectSnap = await transaction.get(projectRef);
    if (!projectSnap.exists()) throw new Error('Project not found');

    const memberSnap = await transaction.get(memberRef);
    if (memberSnap.exists()) throw new Error('Already a member');

    transaction.set(memberRef, { role, joinedAt: serverTimestamp() });
    transaction.update(projectRef, { memberCount: increment(1) });
  });
}
```

## Firestore security rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(projectId) {
      return isAuthenticated() &&
        exists(/databases/$(database)/documents/projects/$(projectId)/members/$(request.auth.uid));
    }

    function getRole(projectId) {
      return get(/databases/$(database)/documents/projects/$(projectId)/members/$(request.auth.uid)).data.role;
    }

    // Projects
    match /projects/{projectId} {
      // Anyone authenticated can create
      allow create: if isAuthenticated();

      // Members can read
      allow read: if isOwner(projectId);

      // Admin+ can update
      allow update: if isOwner(projectId) && getRole(projectId) in ['owner', 'admin'];

      // Only owners can delete
      allow delete: if isOwner(projectId) && getRole(projectId) == 'owner';

      // Subcollections
      match /members/{userId} {
        allow read: if isOwner(projectId);
        allow create: if isOwner(projectId) && getRole(projectId) in ['owner', 'admin'];
        allow update: if isOwner(projectId) &&
          (getRole(projectId) in ['owner', 'admin'] || userId == request.auth.uid);
        allow delete: if isOwner(projectId) && getRole(projectId) in ['owner', 'admin'];
      }

      match /tasks/{taskId} {
        allow read: if isOwner(projectId);
        allow create: if isOwner(projectId);
        allow update: if isOwner(projectId);
        allow delete: if isOwner(projectId) && getRole(projectId) in ['owner', 'admin'];
      }
    }
  }
}
```

### Security rules testing

```ts
import { assertSucceeds, assertFails } from '@firebase/rules-unit-testing';

test('member can read project', async () => {
  const db = getAuthedFirestore(userId);
  await assertSucceeds(db.doc('projects/proj1').get());
});

test('non-member cannot read project', async () => {
  const db = getAuthedFirestore(unrelatedUserId);
  await assertFails(db.doc('projects/proj1').get());
});

test('viewer cannot delete project', async () => {
  const db = getAuthedFirestore(viewerId);
  await assertFails(db.doc('projects/proj1').delete());
});
```

## Firebase Authentication

```ts
import { signInWithPopup, GoogleAuthProvider, signOut, onAuthStateChanged } from 'firebase/auth';

// Google sign-in
const provider = new GoogleAuthProvider();
await signInWithPopup(auth, provider);

// Email/password sign-up
import { createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth';
await createUserWithEmailAndPassword(auth, email, password);
await signInWithEmailAndPassword(auth, email, password);

// Auth state listener
onAuthStateChanged(auth, (user) => {
  if (user) {
    setUser({ uid: user.uid, email: user.email, name: user.displayName });
  } else {
    setUser(null);
  }
});

// Sign out
await signOut(auth);
```

## Cloud Storage (file uploads)

```ts
import { ref, uploadBytes, getDownloadURL } from 'firebase/storage';

async function uploadProjectImage(projectId: string, file: File) {
  const fileRef = ref(storage, `projects/${projectId}/${file.name}`);
  await uploadBytes(fileRef, file);
  const url = await getDownloadURL(fileRef);
  return url;
}
```

### Storage security rules

```javascript
rules_version = '2';
service cloud.storage {
  match /b/{bucket}/o {
    // Project images: only members can upload, anyone with URL can read
    match /projects/{projectId}/{allPaths=**} {
      allow read: if true;  // Public read (images are served via public URL)
      allow write: if request.auth != null && request.resource.size < 5 * 1024 * 1024;  // 5MB max
    }
  }
}
```

## Cloud Functions

### HTTP callable (called from client)

```ts
import { onCall } from 'firebase-functions/v2/https';

export const inviteMember = onCall({ cors: true }, async (request) => {
  const { projectId, email, role } = request.data;
  const uid = request.auth?.uid;

  if (!uid) throw new HttpsError('unauthenticated', 'Must be signed in');

  const projectDoc = await db.doc(`projects/${projectId}`).get();
  const memberDoc = await db.doc(`projects/${projectId}/members/${uid}`).get();

  if (!memberDoc.exists || !['owner', 'admin'].includes(memberDoc.data().role)) {
    throw new HttpsError('permission-denied', 'Not authorized to invite');
  }

  const inviteLink = await createInviteLink(projectId, email, role);
  await sendInviteEmail(email, inviteLink);

  return { success: true };
});
```

### Firestore triggers

```ts
import { onDocumentCreated } from 'firebase-functions/v2/firestore';

export const onTaskCreated = onDocumentCreated(
  'projects/{projectId}/tasks/{taskId}',
  async (event) => {
    const task = event.data?.data();
    if (!task) return;

    if (task.assigneeId) {
      await sendNotification(task.assigneeId, {
        title: 'New task assigned',
        body: task.title,
      });
    }
  }
);
```

### Scheduled functions

```ts
import { onSchedule } from 'firebase-functions/v2/scheduler';

export const cleanupExpiredInvites = onSchedule('every day 02:00', async () => {
  const expired = await db
    .collection('invites')
    .where('expiresAt', '<', Timestamp.now())
    .get();

  const batch = db.batch();
  expired.docs.forEach(doc => batch.delete(doc.ref));
  await batch.commit();
});
```

## Quality checklist

- [ ] Firebase config uses environment variables (not hardcoded)
- [ ] Emulator suite used for local development
- [ ] Firestore data model follows denormalization principles
- [ ] Counter fields updated with FieldValue.increment
- [ ] Real-time listeners cleaned up on unmount (useEffect cleanup)
- [ ] Security rules test with rules-unit-testing
- [ ] Security rules validate authentication and authorization
- [ ] Storage rules limit file size and types
- [ ] Cloud Functions validate input and auth
- [ ] Composite indexes defined in firestore.indexes.json
- [ ] Batch writes for multi-document atomic operations
- [ ] Transactions for read-then-write operations

## Anti-patterns I avoid

- Treating Firestore like SQL — it's a document database, denormalize
- Missing security rules — data is publicly readable/writable by default
- Not cleaning up onSnapshot listeners — memory leaks
- Using Firestore for full-text search — use Algolia or Elastic
- Storing large files in Firestore — use Cloud Storage
- Nested listener callbacks — use async/await patterns
- Client-side business logic that should be in Cloud Functions — security rules are not business logic
- Not using emulators for development — real Firestore costs money on every read/write