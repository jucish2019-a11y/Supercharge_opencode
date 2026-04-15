---
name: email
description: Implement transactional email — templates, sending infrastructure (Resend, SendGrid), email verification, password reset, notification emails, and rate limiting
---

## What I do

I implement transactional email systems for web applications:

- **Email infrastructure** — Resend, SendGrid, AWS SES — choosing and configuring
- **Email templates** — React Email, MJML, HTML design patterns
- **Sending patterns** — Transactional, notification, digest, batch
- **Email verification** — Signup confirmation, email change
- **Password reset** — Secure token generation, expiry, flow
- **Delivery** — Webhook tracking, bounce handling, rate limiting

## When to use me

Use this skill when:
- Setting up transactional email for a new application
- Designing email templates that work across clients
- Implementing email verification during signup
- Building password reset functionality
- Choosing between Resend, SendGrid, and AWS SES
- Handling email delivery tracking and bounces

## Provider selection

| Provider | Best for | Pricing | DX |
|----------|----------|---------|-----|
| **Resend** | Modern apps, best DX, React Email | Free 3K/mo, then $20/100K | Excellent |
| **SendGrid** | High volume, established | Free 100/day, then $15/mo | Good |
| **AWS SES** | Cheapest at scale, AWS shops | $0.10/1000 | Poor |
| **Postmark** | Reliable delivery, good analytics | Free 100/mo, then $15/mo | Good |
| **Mailgun** | Developer-focused, flexible | Free 5K/mo, then $35/mo | Good |

**Recommendation**: Start with Resend for best DX and React Email support. Switch to SES only at very high volume (>100K/mo).

## Resend setup

```bash
npm install resend react-email @react-email/components
```

```ts
// lib/email.ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);

export const FROM_ADDRESS = 'App Name <noreply@appname.com>';
```

## React Email templates

### Template structure

```
emails/
  components/
    layout.tsx          ← Base layout (header, footer, styles)
    button.tsx          ← Styled CTA button
    heading.tsx         ← Consistent heading styles
  verification.tsx      ← Email verification
  password-reset.tsx    ← Password reset
  welcome.tsx           ← Welcome / onboarding
  notification.tsx      ← Notification template
  invoice.tsx           ← Invoice / receipt
```

### Base layout

```tsx
// emails/components/layout.tsx
import { Html, Head, Body, Container, Section, Text } from '@react-email/components';

type LayoutProps = {
  children: React.ReactNode;
  previewText: string;
};

export function Layout({ children, previewText }: LayoutProps) {
  return (
    <Html>
      <Head />
      <Body style={bodyStyle}>
        <Container style={containerStyle}>
          <Section style={headerStyle}>
            <Text style={logoStyle}>App Name</Text>
          </Section>
          <Section style={contentStyle}>
            {children}
          </Section>
          <Section style={footerStyle}>
            <Text style={footerTextStyle}>
              © 2026 App Name. All rights reserved.
            </Text>
            <Text style={footerTextStyle}>
              <a href="https://appname.com/settings/notifications" style={linkStyle}>
                Unsubscribe
              </a>
              {' · '}
              <a href="https://appname.com/settings" style={linkStyle}>
                Preferences
              </a>
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  );
}

const bodyStyle = {
  backgroundColor: '#f6f6f6',
  fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
};

const containerStyle = {
  maxWidth: '600px',
  margin: '0 auto',
  backgroundColor: '#ffffff',
  borderRadius: '8px',
  overflow: 'hidden',
};

const headerStyle = {
  backgroundColor: '#171717',
  padding: '32px 40px',
};

const logoStyle = {
  color: '#ffffff',
  fontSize: '24px',
  fontWeight: '700',
  margin: '0',
};

const contentStyle = {
  padding: '40px',
};

const footerStyle = {
  padding: '24px 40px',
  borderTop: '1px solid #e5e5e5',
};

const footerTextStyle = {
  color: '#737373',
  fontSize: '12px',
  lineHeight: '18px',
  margin: '0 0 4px',
};

const linkStyle = {
  color: '#3B82F6',
  textDecoration: 'underline',
};
```

### Email verification template

```tsx
// emails/verification.tsx
import { Heading, Text, Button, Hr } from '@react-email/components';
import { Layout } from './components/layout';

type VerificationEmailProps = {
  userName: string;
  verificationUrl: string;
};

export function VerificationEmail({ userName, verificationUrl }: VerificationEmailProps) {
  return (
    <Layout previewText="Verify your email address">
      <Heading style={headingStyle}>Verify your email</Heading>
      <Text style={textStyle}>
        Hi {userName},
      </Text>
      <Text style={textStyle}>
        Please verify your email address to complete your account setup.
        This link expires in 24 hours.
      </Text>
      <Button style={buttonStyle} href={verificationUrl}>
        Verify email address
      </Button>
      <Hr style={hrStyle} />
      <Text style={subTextStyle}>
        If you didn't create an account, you can safely ignore this email.
      </Text>
      <Text style={subTextStyle}>
        If the button doesn't work, copy and paste this link into your browser:
      </Text>
      <Text style={linkTextStyle}>{verificationUrl}</Text>
    </Layout>
  );
}

const headingStyle = { fontSize: '24px', fontWeight: '600', margin: '0 0 24px' };
const textStyle = { fontSize: '16px', lineHeight: '24px', color: '#171717', margin: '0 0 16px' };
const subTextStyle = { fontSize: '14px', lineHeight: '20px', color: '#737373', margin: '0 0 8px' };
const linkTextStyle = { fontSize: '14px', lineHeight: '20px', color: '#3B82F6', margin: '4px 0 0' };
const buttonStyle = {
  backgroundColor: '#171717',
  borderRadius: '6px',
  color: '#ffffff',
  fontSize: '16px',
  fontWeight: '600',
  textDecoration: 'none',
  padding: '12px 24px',
  display: 'inline-block',
};
const hrStyle = { borderColor: '#e5e5e5', margin: '24px 0' };
```

### Send email function

```ts
// lib/email.ts
import { resend, FROM_ADDRESS } from './resend';
import { VerificationEmail } from '../emails/verification';

export async function sendVerificationEmail(email: string, userName: string, verificationUrl: string) {
  const { error } = await resend.emails.send({
    from: FROM_ADDRESS,
    to: email,
    subject: 'Verify your email address',
    react: VerificationEmail({ userName, verificationUrl }),
  });

  if (error) {
    console.error('Failed to send verification email:', error);
    throw new Error('Failed to send verification email');
  }
}
```

## Email verification flow

### Token generation and storage

```ts
import { crypto } from 'node: crypto';

async function createVerificationToken(userId: string): Promise<string> {
  const token = crypto.randomBytes(32).toString('hex');
  const expiresAt = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 hours

  await db.verificationToken.create({
    data: {
      token,
      userId,
      expiresAt,
      type: 'EMAIL_VERIFICATION',
    },
  });

  return token;
}

async function verifyEmail(token: string) {
  const verificationToken = await db.verificationToken.findUnique({
    where: { token },
  });

  if (!verificationToken) throw new Error('Invalid token');
  if (verificationToken.expiresAt < new Date()) throw new Error('Token expired');
  if (verificationToken.usedAt) throw new Error('Token already used');

  await db.$transaction([
    db.user.update({
      where: { id: verificationToken.userId },
      data: { emailVerified: true },
    }),
    db.verificationToken.update({
      where: { token },
      data: { usedAt: new Date() },
    }),
  ]);
}
```

### API routes

```ts
// POST /api/auth/verify - Send verification email
app.post('/api/auth/verify', async (req, res) => {
  const user = req.user;
  if (user.emailVerified) {
    return res.json({ message: 'Email already verified' });
  }

  const token = await createVerificationToken(user.id);
  const verificationUrl = `${process.env.APP_URL}/verify?token=${token}`;

  await sendVerificationEmail(user.email, user.name, verificationUrl);

  res.json({ message: 'Verification email sent' });
});

// POST /api/auth/verify/:token - Verify email
app.post('/api/auth/verify/:token', async (req, res) => {
  try {
    await verifyEmail(req.params.token);
    res.json({ message: 'Email verified successfully' });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

## Password reset flow

### Token generation

```ts
async function createPasswordResetToken(email: string): Promise<string> {
  const user = await db.user.findUnique({ where: { email } });
  if (!user) {
    // Don't reveal if email exists — always return success
    return 'sent';
  }

  const token = crypto.randomBytes(32).toString('hex');
  const expiresAt = new Date(Date.now() + 60 * 60 * 1000); // 1 hour

  await db.verificationToken.create({
    data: {
      token,
      userId: user.id,
      expiresAt,
      type: 'PASSWORD_RESET',
    },
  });

  const resetUrl = `${process.env.APP_URL}/reset-password?token=${token}`;
  await sendPasswordResetEmail(user.email, user.name, resetUrl);

  return 'sent';
}
```

### Password reset email template

```tsx
export function PasswordResetEmail({ userName, resetUrl }: { userName: string; resetUrl: string }) {
  return (
    <Layout previewText="Reset your password">
      <Heading style={headingStyle}>Reset your password</Heading>
      <Text style={textStyle}>Hi {userName},</Text>
      <Text style={textStyle}>
        We received a request to reset your password. Click the button below
        to create a new password. This link expires in 1 hour.
      </Text>
      <Button style={buttonStyle} href={resetUrl}>
        Reset password
      </Button>
      <Hr style={hrStyle} />
      <Text style={subTextStyle}>
        If you didn't request a password reset, you can safely ignore this email.
        Your password will not be changed.
      </Text>
    </Layout>
  );
}
```

### API routes

```ts
// POST /api/auth/forgot-password
app.post('/api/auth/forgot-password', async (req, res) => {
  const { email } = req.body;
  await createPasswordResetToken(email);
  // Always return success to prevent email enumeration
  res.json({ message: 'If an account exists with this email, a reset link has been sent.' });
});

// POST /api/auth/reset-password
app.post('/api/auth/reset-password', async (req, res) => {
  const { token, password } = req.body;

  const resetToken = await db.verificationToken.findUnique({ where: { token } });
  if (!resetToken) return res.status(400).json({ error: 'Invalid token' });
  if (resetToken.expiresAt < new Date()) return res.status(400).json({ error: 'Token expired' });
  if (resetToken.usedAt) return res.status(400).json({ error: 'Token already used' });

  const hashedPassword = await hash(password);
  await db.$transaction([
    db.user.update({
      where: { id: resetToken.userId },
      data: { password: hashedPassword },
    }),
    db.verificationToken.update({
      where: { token },
      data: { usedAt: new Date() },
    }),
  ]);

  res.json({ message: 'Password reset successfully' });
});
```

## Email template best practices

### HTML email rules

```
Styling:
  - Use inline styles (email clients strip <style> tags)
  - Use tables for layout (not flexbox or grid)
  - Use px for font sizes (not rem or em)
  - Maximum width: 600px
  - Use web-safe fonts: Arial, Helvetica, Georgia, Times New Roman
  - Font stack: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif

Images:
  - Host images on CDN (never attachments)
  - Always include alt text
  - Use absolute URLs (never relative)
  - Max image width: 600px
  - Provide a text alternative for every image

Buttons:
  - Use <a> with inline styles (not <button>)
  - Add padding for tap targets (minimum 44px height)
  - Include a plain text link below the button (for clients that don't render HTML)

Subject lines:
  - Under 50 characters for mobile
  - No emoji in transactional subjects
  - Clear and specific: "Verify your email" not "Action required"
  - Preview text: complements the subject line

Links:
  - Use absolute URLs (never relative)
  - Don't use URL shorteners (triggers spam filters)
  - Include UTM parameters for tracking
```

### Testing

```bash
# Resend provides a test API key that sends to your email
RESEND_API_KEY=re_test_...  # Test mode
```

```ts
// Test email rendering
import { render } from '@react-email/components';
import { VerificationEmail } from './emails/verification';

const html = render(<VerificationEmail userName="John" verificationUrl="https://..." />);
console.log(html); // Preview the rendered HTML
```

## Rate limiting

```ts
import { Redis } from 'ioredis';
const redis = new Redis(process.env.REDIS_URL);

async function checkEmailRateLimit(userId: string, type: string): Promise<boolean> {
  const key = `email-rate:${type}:${userId}`;
  const current = await redis.incr(key);
  
  if (current === 1) {
    await redis.expire(key, 3600); // 1 hour window
  }
  
  const limits: Record<string, number> = {
    verification: 3,    // 3 verification emails per hour
    password_reset: 3,  // 3 password resets per hour
    notification: 10,   // 10 notification emails per hour
  };
  
  return current <= (limits[type] || 5);
}

// Usage
app.post('/api/auth/verify', async (req, res) => {
  const allowed = await checkEmailRateLimit(req.user.id, 'verification');
  if (!allowed) {
    return res.status(429).json({ error: 'Too many emails. Please try again later.' });
  }
  // ... send verification email
});
```

## Email types and when to send

| Type | Trigger | Frequency | Template |
|------|---------|-----------|----------|
| Email verification | Signup | Once | verification.tsx |
| Password reset | User request | On demand (rate limited) | password-reset.tsx |
| Welcome | First login | Once | welcome.tsx |
| Invoice/receipt | Payment successful | Per payment | invoice.tsx |
| Payment failed | Payment declined | Per attempt (max 3) | payment-failed.tsx |
| Subscription ending | 7 days before expiry | Once | subscription-ending.tsx |
| Weekly digest | Scheduled | Weekly | digest.tsx |
| Notification | User action | On event (rate limited) | notification.tsx |
| Team invite | Admin invites | On demand | team-invite.tsx |

## Quality checklist

- [ ] Email provider configured (Resend/SendGrid) with domain verification
- [ ] Templates use React Email components (or tested MJML)
- [ ] Inline styles only (no <style> tags — email clients strip them)
- [ ] Max width 600px, responsive layout
- [ ] Plain text fallback for every HTML email
- [ ] Email verification flow: token generation, 24-hour expiry, one-time use
- [ ] Password reset flow: token generation, 1-hour expiry, one-time use
- [ ] Rate limiting on all outgoing emails (prevent abuse)
- [ ] Unsubscribe link in every email (CAN-SPAM compliance)
- [ ] UTM parameters on all links for tracking
- [ ] Email previews tested in Gmail, Outlook, Apple Mail, mobile
- [ ] Dark mode styles included
- [ ] DKIM, SPF, DMARC DNS records configured
- [ ] Webhook endpoint for delivery tracking (bounces, complaints)
- [ ] Token invalidation after use (prevent replay attacks)

## Anti-patterns I avoid

- Sending emails from personal addresses (noreply@ or hello@ with custom domain)
- Using <style> tags — most email clients strip them
- Using flexbox or grid — many email clients don't support them
- Relative URLs — they break in email
- Revealing if an email exists (forgot-password always returns "sent")
- Rate-limiting without email type limits (3 verification emails, not 3 total)
- Storing passwords in reset tokens (store a random token, hash it in DB)
- Not invalidating tokens after use (replay attacks)
- Using the same token for email verification and password reset
- Sending HTML-only emails (always include plain text alternative)
- Not setting up DKIM/SPF/DMARC (emails go to spam)