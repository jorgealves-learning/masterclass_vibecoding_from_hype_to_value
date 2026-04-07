---
paths:
  - "src/**/*.ts"
  - "src/**/*.tsx"
---

# Security & PII Safety Skills

> Critical skill for GDPR compliance and secure development practices

---

## Purpose

This skill ensures all code adheres to security best practices and GDPR requirements for handling Personally Identifiable Information (PII). Security is a foundational requirement per AGENTS.md §1.4.

---

## PII Identification

### What Constitutes PII

| Category | Examples | Risk Level |
|----------|----------|------------|
| Direct Identifiers | Email, phone, SSN, passport | Critical |
| Quasi-Identifiers | IP address, device ID, cookies | High |
| Sensitive Data | Health info, financial data, biometrics | Critical |
| Behavioral Data | Location history, browsing patterns | High |
| Authentication | Passwords, tokens, API keys | Critical |

### Recognizing PII in Code

```typescript
// ❌ PII fields to protect
type User = {
  email: string;           // PII: Direct identifier
  phoneNumber: string;     // PII: Direct identifier
  ipAddress: string;       // PII: Quasi-identifier
  dateOfBirth: Date;       // PII: Sensitive
  socialSecurityNumber: string; // PII: Critical
  creditCardNumber: string;     // PII: Critical (also PCI-DSS)
};
```

---

## Logging Safety

### Never Log PII

```typescript
// ❌ DANGEROUS: Logging PII
console.log('User login:', { email: user.email, ip: request.ip });
console.log(`Password reset for ${user.email}`);
logger.info('Order placed', { customerEmail, creditCard });

// ✅ SAFE: Log only non-identifying data
console.log('User login:', { userId: user.id, timestamp: Date.now() });
console.log(`Password reset requested`, { userId: user.id });
logger.info('Order placed', { orderId, userId, timestamp: Date.now() });
```

### Safe Logging Pattern

```typescript
import { z } from 'zod';

// Define what's safe to log
const SafeLogSchema = z.object({
  userId: z.string().uuid(),
  action: z.string(),
  timestamp: z.number(),
  resourceId: z.string().optional(),
  success: z.boolean(),
});

type SafeLogEntry = z.infer<typeof SafeLogSchema>;

function safeLog(entry: SafeLogEntry): void {
  // Validate before logging to prevent accidental PII
  const validated = SafeLogSchema.parse(entry);
  console.log(JSON.stringify(validated));
}

// Usage
safeLog({
  userId: user.id,
  action: 'LOGIN',
  timestamp: Date.now(),
  success: true,
});
```

---

## Error Handling Without PII Exposure

### Error Messages to Users

```typescript
// ❌ DANGEROUS: Exposing internal details
return NextResponse.json({
  error: `User with email ${email} not found in database users table`,
  stack: error.stack,
});

// ✅ SAFE: Generic error messages
return NextResponse.json({
  error: 'Authentication failed',
  code: 'AUTH_FAILED',
}, { status: 401 });
```

### Internal Error Logging

```typescript
export async function POST(request: NextRequest): Promise<NextResponse> {
  try {
    // ... operation
  } catch (error) {
    // Log safely for debugging (internal only)
    console.error('Operation failed:', {
      errorType: error instanceof Error ? error.name : 'Unknown',
      errorMessage: error instanceof Error ? error.message : 'Unknown error',
      requestId: request.headers.get('x-request-id'),
      timestamp: new Date().toISOString(),
      // NEVER: email, userId, tokens, passwords
    });

    // Return safe error to client
    return NextResponse.json(
      { error: 'Operation failed', code: 'INTERNAL_ERROR' },
      { status: 500 }
    );
  }
}
```

---

## Environment Variable Security

### Secure Environment Configuration

Per AGENTS.md §4.3, create `/src/lib/env.ts`:

```typescript
import { z } from 'zod';

const envSchema = z.object({
  // Database
  DATABASE_URL: z.string().url().startsWith('postgresql://'),
  
  // Authentication
  NEXTAUTH_SECRET: z.string().min(32, 'Secret must be at least 32 characters'),
  NEXTAUTH_URL: z.string().url(),
  
  // API Keys (never expose to client)
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  
  // Public vars (safe for client)
  NEXT_PUBLIC_APP_URL: z.string().url(),
});

export type Env = z.infer<typeof envSchema>;

function validateEnv(): Env {
  const parsed = envSchema.safeParse(process.env);
  
  if (!parsed.success) {
    console.error('❌ Invalid environment variables');
    console.error(parsed.error.flatten().fieldErrors);
    throw new Error('Environment validation failed');
  }
  
  return parsed.data;
}

export const env = validateEnv();
```

### Never Expose Secrets to Client

```typescript
// ❌ DANGEROUS: Exposing secrets
// This will be bundled and sent to browser!
const apiKey = process.env.STRIPE_SECRET_KEY;

// ✅ SAFE: Only NEXT_PUBLIC_ vars are client-safe
const appUrl = process.env.NEXT_PUBLIC_APP_URL;

// ✅ SAFE: Server-only access
// In Route Handlers or Server Components only
import { env } from '@/lib/env';
const stripeKey = env.STRIPE_SECRET_KEY;
```

---

## Input Validation & Sanitization

### SQL Injection Prevention

```typescript
// ❌ DANGEROUS: String interpolation in queries
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ SAFE: Parameterized queries
const user = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);

// ✅ SAFE: ORM with type safety
const user = await prisma.user.findUnique({
  where: { email },
});
```

### XSS Prevention

```typescript
// ❌ DANGEROUS: Rendering unsanitized HTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ SAFE: React automatically escapes
<div>{userInput}</div>

// ✅ SAFE: If HTML is needed, sanitize first
import DOMPurify from 'dompurify';

const sanitizedHtml = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
  ALLOWED_ATTR: [],
});
<div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />
```

---

## Authentication Security

### Token Handling

```typescript
// ❌ DANGEROUS: Storing tokens insecurely
localStorage.setItem('authToken', token);
document.cookie = `token=${token}`;

// ✅ SAFE: HttpOnly cookies (set from server)
// In Route Handler:
const response = NextResponse.json({ success: true });
response.cookies.set({
  name: 'session',
  value: sessionToken,
  httpOnly: true,      // Not accessible via JavaScript
  secure: true,        // HTTPS only
  sameSite: 'strict',  // CSRF protection
  maxAge: 60 * 60 * 24, // 24 hours
  path: '/',
});
return response;
```

### Password Handling

```typescript
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;

// ✅ SAFE: Hashing passwords
async function hashPassword(password: string): Promise<string> {
  return hash(password, SALT_ROUNDS);
}

// ✅ SAFE: Verifying passwords
async function verifyPassword(password: string, hashedPassword: string): Promise<boolean> {
  return compare(password, hashedPassword);
}

// ❌ NEVER: Store plain text passwords
// ❌ NEVER: Log passwords (even hashed)
// ❌ NEVER: Send passwords in response
```

---

## Data Minimization

### Collect Only What's Needed

```typescript
// ❌ OVER-COLLECTION: Storing unnecessary PII
const userSchema = z.object({
  email: z.string().email(),
  password: z.string(),
  name: z.string(),
  dateOfBirth: z.date(),        // Do we need this?
  phoneNumber: z.string(),       // Do we need this?
  address: z.string(),           // Do we need this?
  socialSecurityNumber: z.string(), // Almost never needed!
});

// ✅ MINIMAL: Only required fields
const userSchema = z.object({
  email: z.string().email(),
  password: z.string(),
  name: z.string(),
  // Add optional fields only when business requirement exists
});
```

### Select Only Needed Fields

```typescript
// ❌ DANGEROUS: Fetching all fields
const user = await prisma.user.findUnique({
  where: { id },
});
return NextResponse.json(user); // Exposes passwordHash, etc.

// ✅ SAFE: Select only needed fields
const user = await prisma.user.findUnique({
  where: { id },
  select: {
    id: true,
    name: true,
    email: true,
    // Explicitly omit: passwordHash, tokens, etc.
  },
});
return NextResponse.json(user);
```

---

## GDPR Compliance Patterns

### Right to Access (Data Export)

```typescript
export async function exportUserData(userId: string): Promise<UserDataExport> {
  const [user, orders, preferences] = await Promise.all([
    prisma.user.findUnique({ where: { id: userId } }),
    prisma.order.findMany({ where: { userId } }),
    prisma.preference.findMany({ where: { userId } }),
  ]);

  return {
    personalInfo: {
      name: user?.name,
      email: user?.email,
      createdAt: user?.createdAt,
    },
    orders: orders.map(o => ({
      id: o.id,
      date: o.createdAt,
      total: o.total,
    })),
    preferences,
    exportedAt: new Date().toISOString(),
  };
}
```

### Right to Erasure (Data Deletion)

```typescript
export async function deleteUserData(userId: string): Promise<void> {
  await prisma.$transaction([
    // Delete related data first
    prisma.preference.deleteMany({ where: { userId } }),
    prisma.session.deleteMany({ where: { userId } }),
    prisma.order.updateMany({
      where: { userId },
      data: { 
        // Anonymize instead of delete for business records
        customerName: '[DELETED]',
        customerEmail: '[DELETED]',
      },
    }),
    // Finally delete user
    prisma.user.delete({ where: { id: userId } }),
  ]);

  // Log deletion (without PII)
  console.log('User data deleted:', { 
    userId, 
    timestamp: new Date().toISOString() 
  });
}
```

---

## Security Headers

### Next.js Security Headers Configuration

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on',
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload',
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN',
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()',
  },
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};
```

---

## Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'), // 10 requests per 10 seconds
  analytics: true,
});

export async function POST(request: NextRequest): Promise<NextResponse> {
  // Use a non-PII identifier for rate limiting
  const identifier = request.headers.get('x-forwarded-for') ?? 'anonymous';
  const { success, limit, remaining } = await ratelimit.limit(identifier);

  if (!success) {
    return NextResponse.json(
      { error: 'Too many requests' },
      { 
        status: 429,
        headers: {
          'X-RateLimit-Limit': limit.toString(),
          'X-RateLimit-Remaining': remaining.toString(),
        },
      }
    );
  }

  // Continue with request handling
}
```

---

## Security Checklist

Before merging any code:

- [ ] No PII in logs or error messages
- [ ] All inputs validated with Zod at boundaries
- [ ] No secrets exposed to client bundle
- [ ] Passwords properly hashed (bcrypt, 12+ rounds)
- [ ] SQL queries use parameterization
- [ ] User data queries use field selection (not SELECT *)
- [ ] Sensitive cookies are HttpOnly + Secure + SameSite
- [ ] Rate limiting on authentication endpoints
- [ ] Security headers configured
- [ ] No `any` types for external data
