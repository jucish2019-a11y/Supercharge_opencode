---
name: security-audit
description: Scan for security vulnerabilities including secrets, injection, auth issues, and dependency risks
---

## What I do

I perform comprehensive security-focused code audits:

- **Secrets exposure** — API keys, passwords, tokens, private keys in source code
- **Injection attacks** — SQL injection, XSS, command injection, path traversal, LDAP injection
- **Authentication & authorization** — Bypass opportunities, missing auth checks, privilege escalation
- **Input validation** — Missing validation, type confusion, prototype pollution, unsafe deserialization
- **Dependency risks** — Known vulnerable packages, outdated dependencies with CVEs
- **Configuration** — Insecure defaults, CORS misconfiguration, missing rate limiting
- **Web vulnerabilities** — CSRF, clickjacking, open redirects, insecure cookies
- **Infrastructure** — IAM permissions, exposed services, missing encryption

## When to use me

Use this skill when:
- Before deploying to production
- When adding authentication, authorization, or payment features
- After adding new API endpoints or user input handling
- Periodically for security health checks
- When incorporating new dependencies
- After security incidents or bug bounty reports

## OWASP Top 10 checklist

### A01: Broken Access Control

```ts
// VULNERABLE: Missing authorization check
app.get('/api/documents/:id', async (req, res) => {
  const doc = await db.document.findUnique({ where: { id: req.params.id } });
  return res.json(doc); // Anyone can access any document!
});

// SECURE: Verify ownership
app.get('/api/documents/:id', authenticate, async (req, res) => {
  const doc = await db.document.findUnique({
    where: { id: req.params.id },
    include: { owner: true },
  });

  if (!doc) return res.status(404).json({ error: 'Not found' });
  if (doc.ownerId !== req.user.id && req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }

  return res.json(doc);
});
```

### A02: Cryptographic Failures

```ts
// SECURE: Password hashing with Argon2
import { hash, verify } from 'argon2';

const hashedPassword = await hash(password, {
  type: argon2id,
  memoryCost: 65536,
  timeCost: 3,
});

// SECURE: Data at rest encryption
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

function encrypt(text: string, key: Buffer): { iv: string; encrypted: string } {
  const iv = randomBytes(16);
  const cipher = createCipheriv('aes-256-gcm', key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  const authTag = cipher.getAuthTag();
  return { iv: iv.toString('hex'), encrypted: encrypted + authTag.toString('hex') };
}

// SECURE: HTTPS enforcement
// In middleware or reverse proxy:
if (req.headers['x-forwarded-proto'] !== 'https') {
  return res.redirect(301, `https://${req.headers.host}${req.url}`);
}
```

### A03: Injection

```ts
// VULNERABLE: SQL injection
const query = `SELECT * FROM users WHERE email = '${email}'`;

// SECURE: Parameterized queries
const users = await db.$queryRaw`
  SELECT * FROM users WHERE email = ${email}
`;

// SECURE: ORM usage
const user = await db.user.findUnique({ where: { email } });

// VULNERABLE: Command injection
const { exec } = require('child_process');
exec(`convert ${userInput} output.png`); // Dangerous!

// SECURE: Whitelist allowed inputs
const ALLOWED_FORMATS = ['png', 'jpg', 'webp'];
const format = ALLOWED_FORMATS.find(f => f === userInput);
if (!format) throw new Error('Invalid format');
```

### A04: Insecure Design

```ts
// SECURE: Rate limiting
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', apiLimiter);

// SECURE: Business logic validation
async function transferMoney(fromAccount: string, toAccount: string, amount: number) {
  if (amount <= 0) throw new Error('Amount must be positive');
  if (amount > 10000) throw new Error('Amount exceeds daily limit');

  return db.$transaction(async (tx) => {
    const sender = await tx.account.findUnique({ where: { id: fromAccount } });
    if (!sender || sender.balance < amount) {
      throw new Error('Insufficient funds');
    }

    await tx.account.update({
      where: { id: fromAccount },
      data: { balance: { decrement: amount } },
    });

    await tx.account.update({
      where: { id: toAccount },
      data: { balance: { increment: amount } },
    });
  });
}
```

### A05: Security Misconfiguration

```ts
// SECURE: Security headers
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
}));

// SECURE: CORS configuration
import cors from 'cors';

const corsOptions = {
  origin: process.env.ALLOWED_ORIGINS?.split(',') ?? ['http://localhost:3000'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
};

app.use(cors(corsOptions));
```

### A06: Vulnerable and Outdated Components

```bash
# Scan for vulnerabilities
npm audit
npm audit fix

# Check for outdated packages
npm outdated

# Use Snyk for continuous monitoring
npx snyk test
npx snyk monitor

# Check specific package for CVEs
npm audit --audit-level moderate
```

### A07: Identification and Authentication Failures

```ts
// SECURE: Session management
app.use(session({
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
  },
  store: new RedisStore({ client: redisClient }),
}));

// SECURE: Password policy
const PasswordSchema = z.string()
  .min(12, 'Password must be at least 12 characters')
  .regex(/[A-Z]/, 'Must contain uppercase')
  .regex(/[a-z]/, 'Must contain lowercase')
  .regex(/[0-9]/, 'Must contain number')
  .regex(/[^A-Za-z0-9]/, 'Must contain special character');

// SECURE: MFA
import speakeasy from 'speakeasy';

function generateTOTPSecret(): { secret: string; qrCodeUrl: string } {
  const secret = speakeasy.generateSecret({
    name: 'MyApp',
    length: 32,
  });

  const qrCodeUrl = speakeasy.otpauthURL({
    secret: secret.ascii,
    label: 'MyApp',
    issuer: 'MyApp',
  });

  return { secret: secret.base32, qrCodeUrl };
}
```

### A08: Software and Data Integrity Failures

```ts
// SECURE: Dependency integrity
// Use package-lock.json and npm ci in CI/CD
// Verify checksums: npm audit signatures

// SECURE: CI/CD pipeline integrity
// Require signed commits
// Require PR reviews before merge
// Use branch protection rules

// SECURE: Serialization safety
import { z } from 'zod';

const UserInputSchema = z.object({
  name: z.string(),
  age: z.number(),
});

// Never use eval() or new Function()
// Never deserialize untrusted data with JSON.parse without validation
```

### A09: Security Logging and Monitoring Failures

```ts
// SECURE: Audit logging
interface AuditLog {
  timestamp: string;
  userId: string;
  action: string;
  resource: string;
  result: 'success' | 'failure';
  ipAddress: string;
  userAgent: string;
  metadata?: Record<string, unknown>;
}

async function logAuditEvent(event: AuditLog): Promise<void> {
  await db.auditLog.create({ data: event });

  // Alert on suspicious activity
  if (event.result === 'failure' && event.action === 'login') {
    await alertSecurityTeam(`Failed login for user ${event.userId}`);
  }
}
```

### A10: Server-Side Request Forgery (SSRF)

```ts
// VULNERABLE: Unrestricted URL
app.get('/api/fetch', async (req, res) => {
  const response = await fetch(req.query.url as string);
  res.send(await response.text());
});

// SECURE: URL whitelist
const ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com'];

function isAllowedUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    return ALLOWED_DOMAINS.includes(parsed.hostname);
  } catch {
    return false;
  }
}

app.get('/api/fetch', async (req, res) => {
  const url = req.query.url as string;
  if (!isAllowedUrl(url)) {
    return res.status(400).json({ error: 'URL not allowed' });
  }

  const response = await fetch(url);
  res.send(await response.text());
});
```

## Secret scanning

```bash
# Manual checks
grep -r "sk-[a-zA-Z0-9]{48}" .           # OpenAI keys
grep -r "AKIA[0-9A-Z]{16}" .             # AWS access keys
grep -r "ghp_[a-zA-Z0-9]{36}" .          # GitHub tokens
grep -r "private_key" .                  # Private keys
grep -r "password\|secret\|token" . --include="*.env*" --include="*.json"

# Automated tools
# - GitHub secret scanning
# - GitLeaks: gitleaks detect --source .
# - TruffleHog: trufflehog filesystem .
# - Detect Secrets: detect-secrets scan
```

## Output format

```
[SEVERITY] category — short description
  file:line
  Attack vector: how this could be exploited
  Remediation: specific fix
```

Example:
```
[CRITICAL] secrets-exposure — Hardcoded AWS access key in source code
  src/config.ts:15
  Attack vector: Anyone with code access can use this key to access AWS resources
  Remediation: Move to environment variable: process.env.AWS_ACCESS_KEY_ID
```

## Guidelines

- Always flag hardcoded credentials as CRITICAL
- Assume all user input is malicious
- Don't just find issues — provide actionable fixes with code examples
- Consider the full attack chain, not just individual weaknesses
- Test fixes by attempting the same attack after remediation
- Document security decisions and trade-offs

## Security headers checklist

- [ ] `Content-Security-Policy` configured
- [ ] `X-Frame-Options: DENY` or `SAMEORIGIN`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `Referrer-Policy` set
- [ ] `Permissions-Policy` configured
- [ ] `Strict-Transport-Security` (HSTS) enabled
- [ ] Cookies: `httpOnly`, `secure`, `sameSite`
- [ ] CORS properly configured (not `*` in production)

## Anti-patterns I avoid

- Hardcoding any secrets, API keys, or credentials in source code
- Trusting client-side validation as the only validation layer
- Using `eval()`, `new Function()`, or `JSON.parse` on untrusted data
- Storing passwords in plaintext or weak hashing (MD5, SHA1)
- Not using parameterized queries for database access
- Missing authentication on sensitive endpoints
- Allowing CORS `*` in production
- Not implementing rate limiting on auth endpoints
- Logging sensitive data (passwords, tokens, PII)
- Using default credentials or weak passwords
- Not validating redirect URLs (open redirect vulnerability)
- Missing security headers
- Not updating dependencies regularly
- Using HTTP instead of HTTPS in production