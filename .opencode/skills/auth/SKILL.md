---
name: auth
description: Implement authentication and authorization — OAuth2, session management, JWT, RBAC, SSO, and security patterns for production applications
---

## What I do

I implement authentication and authorization systems:

- **Authentication** — Verify who the user is (login, signup, password reset)
- **Authorization** — Verify what the user can do (roles, permissions, RBAC, ABAC)
- **Session management** — Cookies, JWTs, tokens, refresh rotation
- **OAuth2 / social login** — Google, GitHub, Apple, Microsoft sign-in
- **SSO** — SAML, OIDC for enterprise single sign-on
- **Security patterns** — CSRF protection, rate limiting, password hashing, MFA

## When to use me

Use this skill when:
- Adding login/signup to an application
- Implementing OAuth2 social sign-in
- Building role-based access control (RBAC)
- Setting up session management or JWT authentication
- Adding multi-factor authentication (MFA)
- Implementing SSO for enterprise customers

## Auth architecture decision tree

```
Need authentication?
├── Simple app, few users?
│   └── Email + password with sessions
├── Consumer app, want low friction?
│   └── OAuth2 social login (Google, GitHub)
├── B2B SaaS, enterprise customers?
│   └── SSO (SAML/OIDC) + RBAC
├── API-first, microservices?
│   └── JWT with refresh tokens
└── Want to ship fast?
    └── Auth provider (Clerk, Auth0, Supabase Auth, NextAuth.js)
```

## Session vs JWT

### Server-side sessions (recommended for web apps)

```
Login flow:
1. User submits email + password
2. Server validates credentials
3. Server creates session in database/store
4. Server sends session ID in httpOnly cookie
5. On subsequent requests: cookie → session lookup → user

Session store:
  Development: in-memory
  Production: Redis (sub-ms lookup) or database

Cookie settings:
  httpOnly: true       (JS can't read it — XSS protection)
  secure: true         (HTTPS only)
  sameSite: 'lax'      (CSRF protection)
  path: '/'
  maxAge: 7 days       (session lifetime)
  domain: '.app.com'   (shared across subdomains)
```

### JWT (use for APIs and microservices)

```
Login flow:
1. User submits credentials
2. Server validates and creates JWT
3. Server sends access token (short-lived) + refresh token (long-lived)
4. Client stores access token in memory, refresh token in httpOnly cookie
5. Access token in Authorization header for API calls
6. When access token expires, use refresh token to get a new one

Token design:
  Access token: 15 min expiry, contains user ID + roles
  Refresh token: 7 day expiry, stored in DB for revocation
  Both stored: refresh token hash in DB, access token NOT stored

JWT payload:
  {
    "sub": "user_123",         // Subject (user ID)
    "roles": ["admin"],        // Authorization
    "iat": 1713196800,        // Issued at
    "exp": 1713200400         // Expires (15 min)
  }

Rules:
  NEVER put sensitive data in JWT (it's base64, not encrypted)
  NEVER make access tokens long-lived (>15 min)
  ALWAYS validate the signature server-side
  ALWAYS use JWTs over HTTPS
  ALWAYS rotate refresh tokens on use (detect theft)
```

## OAuth2 social login

### Authorization code flow (for web apps)

```
1. Client redirects to provider:
   https://accounts.google.com/o/oauth2/v2/auth?
     client_id=XXX&
     redirect_uri=https://app.com/api/auth/callback/google&
     response_type=code&
     scope=email+profile&
     state=random_csrf_token

2. User authenticates with Google
3. Google redirects back with authorization code:
   https://app.com/api/auth/callback/google?code=AUTH_CODE&state=CSRF_TOKEN

4. Server exchanges code for tokens:
   POST https://oauth2.googleapis.com/token
   { code, client_id, client_secret, redirect_uri, grant_type: "authorization_code" }

5. Server fetches user profile with access token
6. Server finds or creates user in database
7. Server creates session/JWT for the app
```

### PKCE (for mobile/SPA — no client secret)

```
1. Generate code_verifier (random string)
2. code_challenge = SHA256(code_verifier) base64url-encoded
3. Send code_challenge in step 1 instead of relying on client_secret
4. Send code_verifier in step 4 to prove you initiated the request
```

## NextAuth.js / Auth.js pattern

```ts
// lib/auth.ts
import NextAuth from 'next-auth';
import Google from 'next-auth/providers/google';
import GitHub from 'next-auth/providers/github';

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
    }),
  ],
  session: {
    strategy: 'jwt',
    maxAge: 7 * 24 * 60 * 60, // 7 days
  },
  pages: {
    signIn: '/login',
    error: '/login',
  },
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      session.user.id = token.id;
      session.user.role = token.role;
      return session;
    },
  },
});
```

```ts
// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/lib/auth';
export const { GET, POST } = handlers;
```

```ts
// middleware.ts
import { auth } from '@/lib/auth';

export default auth((req) => {
  if (!req.auth && !req.nextUrl.pathname.startsWith('/login')) {
    return Response.redirect(new URL('/login', req.url));
  }
});

export const config = { matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'] };
```

## Role-based access control (RBAC)

### Role model

```ts
type Role = 'owner' | 'admin' | 'member' | 'viewer';

type Permission =
  | 'projects:create'
  | 'projects:read'
  | 'projects:update'
  | 'projects:delete'
  | 'members:invite'
  | 'members:remove'
  | 'billing:read'
  | 'billing:manage';

const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  owner:  ['projects:create', 'projects:read', 'projects:update', 'projects:delete',
           'members:invite', 'members:remove', 'billing:read', 'billing:manage'],
  admin:  ['projects:create', 'projects:read', 'projects:update', 'projects:delete',
           'members:invite', 'members:remove', 'billing:read'],
  member: ['projects:create', 'projects:read', 'projects:update'],
  viewer: ['projects:read'],
};
```

### Permission check

```ts
function hasPermission(role: Role, permission: Permission): boolean {
  return ROLE_PERMISSIONS[role]?.includes(permission) ?? false;
}

// Middleware-style check
function requirePermission(role: Role, permission: Permission) {
  if (!hasPermission(role, permission)) {
    throw new AuthError('Forbidden', 403);
  }
}
```

### Resource-level authorization (ABAC)

RBAC is role-based. ABAC is attribute-based — permissions depend on the resource:

```ts
// Can user edit this project?
async function canEditProject(user: User, project: Project): Promise<boolean> {
  // Organization-level: admin or above
  const orgRole = await getOrgRole(user.id, project.orgId);
  if (['owner', 'admin'].includes(orgRole)) return true;

  // Project-level: project member with edit permission
  const projectRole = await getProjectRole(user.id, project.id);
  if (['lead', 'contributor'].includes(projectRole)) return true;

  return false;
}
```

## Password security

```ts
// NEVER hash passwords yourself — use a proven library
import { hash, verify } from 'argon2';

// Hashing (registration)
async function createUser(email: string, password: string) {
  const hashedPassword = await hash(password, {
    type: argon2id,     // Recommended: resistant to GPU + side-channel
    memoryCost: 65536,  // 64 MB
    timeCost: 3,        // 3 iterations
  });
  return db.user.create({ data: { email, password: hashedPassword } });
}

// Verification (login)
async function login(email: string, password: string) {
  const user = await db.user.findUnique({ where: { email } });
  if (!user) throw new AuthError('Invalid credentials');

  const valid = await verify(user.password, password);
  if (!valid) throw new AuthError('Invalid credentials');

  return createSession(user.id);
}
```

Rules:
- Use Argon2id (preferred) or bcrypt (acceptable). Never MD5, SHA-256, or SHA-512.
- Never log, store, or transmit passwords in plaintext
- Never set maximum password length (DoS via hashing): cap at 1024 chars
- Never return specific error for "user not found" vs "wrong password" — both return "Invalid credentials"
- Rate limit login attempts: 5 per minute per email, 20 per minute per IP

## MFA (Multi-factor authentication)

### TOTP (Time-based One-Time Password)

```
Setup flow:
1. User enables MFA in settings
2. Server generates TOTP secret
3. Server shows QR code (otpauth://totp/...)
4. User scans with authenticator app (Google Authenticator, Authy)
5. User enters first code to verify setup
6. Server marks MFA as enabled, stores secret

Login flow with MFA:
1. User enters email + password → session with mfa_pending: true
2. Server prompts for TOTP code
3. User enters 6-digit code from authenticator app
4. Server verifies code (±1 window for clock drift)
5. Session upgraded to mfa_verified: true
```

### Recovery codes

```
On MFA setup: generate 10 single-use recovery codes
Store: hashed in database (like passwords)
Display: once to the user for safe storage
Usage: user can use a recovery code instead of TOTP if they lose their device
After use: code is consumed and removed from valid codes
```

## CSRF protection

```
For server-side sessions (cookies):
  Use sameSite: 'lax' cookies (protects against most CSRF)
  For state-changing requests: synchronizer token pattern
    1. Server sets CSRF token in cookie
    2. JavaScript reads cookie, includes in request header
    3. Server validates: cookie token === header token

For JWT (Authorization header):
  CSRF is not a concern — tokens aren't sent automatically by browsers
  But: if tokens are stored in cookies, same CSRF risk as sessions
```

## Security checklist

- [ ] Passwords hashed with Argon2id or bcrypt (never plaintext or SHA)
- [ ] Sessions use httpOnly, secure, sameSite=lax cookies
- [ ] JWTs have short expiry (≤15 min for access tokens)
- [ ] Refresh tokens are single-use and rotated
- [ ] OAuth2 state parameter validates on callback
- [ ] Login errors are generic ("Invalid credentials", not "User not found")
- [ ] Rate limiting on auth endpoints (5 req/min per email)
- [ ] CSRF protection on state-changing endpoints
- [ ] MFA available for user accounts
- [ ] Account lockout after 10 failed attempts (time-based, not permanent)
- [ ] Revocation mechanism for sessions/tokens
- [ ] No sensitive data in JWT payloads (not encrypted)
- [ ] Redirect URLs are validated (no open redirects)
- [ ] Environment variables for all secrets (never hardcoded)

## Anti-patterns I avoid

- Storing passwords in plaintext or reversible encryption
- Using JWTs for sessions in server-rendered apps — sessions are simpler and more secure
- Making access tokens long-lived (>15 min)
- Storing refresh tokens in localStorage — use httpOnly cookies
- Specific error messages for auth failures ("Email not registered")
- Rolling your own crypto — always use established libraries
- Rate limiting only by IP (shared IPs in corporate networks) — limit by email too
- Allowing unbounded login attempts — always rate limit
- Trusting client-side auth state alone — always verify server-side