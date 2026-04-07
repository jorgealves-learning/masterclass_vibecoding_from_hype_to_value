---
paths:
  - "vercel.json"
  - "next.config.*"
  - ".env*"
---

# Vercel Deployment Skills

> Skill for deploying Next.js applications on Vercel following AGENTS.md Section 4.0 DevOps standards

---

## Core Principles

Per AGENTS.md:
- **§4.1**: Vercel is the only approved deployment platform
- **§4.2**: Git-driven deployments only (no manual `vercel deploy --prod`)
- **§4.3**: Environment variables must be managed in Vercel project settings

---

## Git-Driven Deployment Workflow

### Branch Strategy

```
main (production)
├── feature/user-auth      → Preview deployment
├── feature/checkout-flow  → Preview deployment
├── fix/login-bug         → Preview deployment
└── chore/update-deps     → Preview deployment
```

### Deployment Triggers

| Branch | Environment | Auto-Deploy |
|--------|-------------|-------------|
| `main` | Production | ✅ Yes |
| Any other | Preview | ✅ Yes |
| None (manual) | Production | ❌ Forbidden* |

*Exception: Declared emergencies only

---

## Environment Variable Management

### Configuration in Vercel Dashboard

```
Vercel Project → Settings → Environment Variables

Variable Types:
├── Production    → Only available in production
├── Preview       → Only available in preview deployments
├── Development   → Only available in local dev (vercel dev)
└── All           → Available everywhere
```

### Required Environment Variables

```bash
# Database
DATABASE_URL=postgresql://...

# Authentication
NEXTAUTH_SECRET=<min-32-char-random-string>
NEXTAUTH_URL=https://your-domain.vercel.app

# API Keys (never expose to client)
STRIPE_SECRET_KEY=sk_live_...
OPENAI_API_KEY=sk-...

# Public variables (safe for client bundle)
NEXT_PUBLIC_APP_URL=https://your-domain.vercel.app
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
```

### Local Development Setup

```bash
# .env.local (gitignored, non-sensitive overrides only)
# NEVER put secrets here - use Vercel CLI to pull them

# Pull environment variables from Vercel
vercel env pull .env.local

# This creates a local copy of your Vercel env vars
```

### Environment Validation

Create `/src/lib/env.ts` (per AGENTS.md §4.3):

```typescript
import { z } from 'zod';

const envSchema = z.object({
  // Database
  DATABASE_URL: z.string().url(),
  
  // Auth
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  
  // External APIs
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  
  // Public (optional validation)
  NEXT_PUBLIC_APP_URL: z.string().url(),
});

export type Env = z.infer<typeof envSchema>;

function validateEnv(): Env {
  const parsed = envSchema.safeParse(process.env);
  
  if (!parsed.success) {
    console.error('❌ Invalid environment variables:');
    console.error(JSON.stringify(parsed.error.flatten().fieldErrors, null, 2));
    throw new Error('Environment validation failed - app cannot start');
  }
  
  return parsed.data;
}

export const env = validateEnv();
```

---

## Vercel Configuration

### `vercel.json` (Optional Customization)

```json
{
  "framework": "nextjs",
  "regions": ["iad1"],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" }
      ]
    }
  ],
  "rewrites": [
    { "source": "/api/:path*", "destination": "/api/:path*" }
  ],
  "redirects": [
    { "source": "/old-page", "destination": "/new-page", "permanent": true }
  ]
}
```

### `next.config.js` for Vercel

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Optimize for Vercel
  output: 'standalone',
  
  // Image optimization
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.example.com',
      },
    ],
  },
  
  // Security headers
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
        ],
      },
    ];
  },
  
  // Environment variables validation at build time
  env: {
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },
};

module.exports = nextConfig;
```

---

## Preview Deployments

### Automatic Preview URLs

Every push to a non-main branch creates:
- `https://<project>-<unique-id>.vercel.app`
- `https://<project>-git-<branch>-<team>.vercel.app`

### Preview Environment Variables

Set preview-specific variables in Vercel:
- Test API keys
- Staging database URLs
- Debug flags

```bash
# In Vercel Dashboard
STRIPE_SECRET_KEY=sk_test_...  # Preview only
DATABASE_URL=postgresql://staging-db/...  # Preview only
```

### Preview Comments

Enable GitHub integration for automatic comments on PRs with preview URLs.

---

## Production Deployment

### Pre-Deployment Checklist

- [ ] All tests pass in CI
- [ ] No TypeScript errors (`tsc --noEmit`)
- [ ] No ESLint errors/warnings
- [ ] Environment variables set in Vercel
- [ ] Database migrations applied
- [ ] PR reviewed and approved

### Deployment Process

```bash
# 1. Merge to main (triggers deployment)
git checkout main
git merge feature/my-feature
git push origin main

# 2. Monitor deployment in Vercel Dashboard
# - Build logs
# - Function logs
# - Analytics

# 3. Verify production
# - Check critical user flows
# - Monitor error rates
# - Check performance metrics
```

---

## Edge Functions & Middleware

### Middleware Configuration

```typescript
// src/middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Auth check example
  const token = request.cookies.get('session');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

### Edge Runtime

```typescript
// Use edge runtime for performance-critical routes
export const runtime = 'edge';

export async function GET(request: Request) {
  // Runs at the edge, closer to users
  return new Response('Hello from the edge!');
}
```

---

## Monitoring & Observability

### Vercel Analytics

```tsx
// src/app/layout.tsx
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Log Drains

Configure log drains in Vercel for:
- Datadog
- LogDNA
- Papertrail
- Custom HTTPS endpoints

### Function Monitoring

```typescript
// Add timing to critical functions
export async function GET(request: NextRequest) {
  const startTime = Date.now();
  
  try {
    const result = await processRequest();
    
    console.log('[API] Request completed', {
      duration: Date.now() - startTime,
      path: request.nextUrl.pathname,
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('[API] Request failed', {
      duration: Date.now() - startTime,
      path: request.nextUrl.pathname,
      error: error instanceof Error ? error.message : 'Unknown',
    });
    throw error;
  }
}
```

---

## Rollback Procedures

### Instant Rollback

```bash
# In Vercel Dashboard:
# Deployments → Select previous deployment → Promote to Production

# Or via CLI:
vercel rollback <deployment-url>
```

### When to Rollback

- Critical bug affecting users
- Performance regression (>50% slower)
- Security vulnerability
- Data corruption risk

---

## Domain Configuration

### Custom Domains

```
Vercel Dashboard → Project → Settings → Domains

Add domains:
├── example.com        → Production
├── www.example.com    → Redirect to example.com
└── staging.example.com → Preview branch
```

### SSL/TLS

Vercel provides automatic SSL certificates:
- Automatic provisioning
- Auto-renewal
- HTTPS redirect

---

## Build Optimization

### Caching

```javascript
// next.config.js
module.exports = {
  // Enable ISR caching
  experimental: {
    isrMemoryCacheSize: 0, // Disable in-memory cache for large sites
  },
};
```

### Build Output

Monitor build output for:
- Bundle size (aim for <100KB initial JS)
- Build time (should be <5 minutes)
- Function size (max 50MB)

```bash
# Check bundle analysis
npx @next/bundle-analyzer

# Or add to next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer(nextConfig);
```

---

## Anti-Patterns to Avoid

### ❌ Manual Production Deployments

```bash
# FORBIDDEN (except emergencies)
vercel deploy --prod

# CORRECT: Push to main
git push origin main
```

### ❌ Secrets in `.env.local`

```bash
# BAD: Secrets in git-ignored file
STRIPE_SECRET_KEY=sk_live_... # in .env.local

# GOOD: Secrets in Vercel Dashboard only
# Pull with: vercel env pull
```

### ❌ Hardcoded URLs

```typescript
// BAD
const apiUrl = 'https://my-app.vercel.app/api';

// GOOD
const apiUrl = process.env.NEXT_PUBLIC_APP_URL + '/api';
```

### ❌ Skipping Preview Testing

```bash
# BAD: Merge without testing preview
git merge feature/untested --no-verify

# GOOD: Always test preview deployment before merge
# 1. Check preview URL
# 2. Run E2E tests against preview
# 3. Get approval
# 4. Merge
```

---

## Deployment Checklist

Before every production deployment:

- [ ] Feature tested locally
- [ ] Preview deployment tested
- [ ] All CI checks pass
- [ ] PR reviewed and approved
- [ ] Database migrations ready
- [ ] Environment variables verified
- [ ] Rollback plan documented
- [ ] Team notified of deployment
