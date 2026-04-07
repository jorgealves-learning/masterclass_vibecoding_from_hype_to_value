# Architecture Documentation
# Next.js TypeScript Applications on Vercel

> This document describes the high-level system architecture for AI agents and developers.
> It complements AGENTS.md (rules) and should be referenced when making structural decisions.

---

## 1.0 System Overview

This is a Next.js application built with the App Router, deployed on Vercel. The system follows a layered architecture with clear separation of concerns between presentation, business logic, data access, and external integrations.

**Core Principles:**
- Server-first rendering with React Server Components
- Edge-optimized for global performance
- Type-safe from database to UI
- Security and PII protection at every layer

---

## 2.0 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           VERCEL EDGE                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Middleware Layer                        │    │
│  │              (Authentication, Rate Limiting)                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        NEXT.JS APP ROUTER                            │
│                                                                      │
│  ┌──────────────────┐    ┌──────────────────┐    ┌───────────────┐  │
│  │   Page Routes    │    │   API Routes     │    │ Server Actions │  │
│  │   (RSC + CSC)    │    │  /(api)/*        │    │ /lib/actions   │  │
│  └────────┬─────────┘    └────────┬─────────┘    └───────┬───────┘  │
│           │                       │                       │          │
│           └───────────────────────┴───────────────────────┘          │
│                                   │                                  │
│                                   ▼                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Service Layer                           │    │
│  │                    /src/lib/services/                        │    │
│  │           (Business Logic, Validation, Orchestration)        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                   │                                  │
│                                   ▼                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Repository Layer                          │    │
│  │                     (Data Access)                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      EXTERNAL SERVICES                               │
│                                                                      │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐                    │
│    │PostgreSQL│    │  Redis   │    │  Auth    │                    │
│    │    DB    │    │  Cache   │    │ Provider │                    │
│    └──────────┘    └──────────┘    └──────────┘                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.0 Layer Definitions

### 3.1 Presentation Layer (`/src/app`, `/src/components`)

**Purpose:** Render UI, handle user interactions, manage client-side state

**What belongs here:**
- Page components (Server Components by default)
- Client Components (for interactivity)
- Layout components
- Loading and error boundaries

**What does NOT belong here:**
- Business logic
- Direct database queries
- External API calls (except via Server Components data fetching)

**Data Flow:**
```
User Request → Page (RSC) → fetch data in RSC → render UI
User Interaction → Client Component → Server Action → Service → Repository
```

### 3.2 API Layer (`/src/app/(api)`)

**Purpose:** HTTP endpoints for external consumers and AJAX requests

**What belongs here:**
- Request validation (Zod schemas)
- Response formatting
- HTTP status code decisions
- Rate limiting enforcement

**What does NOT belong here:**
- Business logic (delegate to Service Layer)
- Direct database queries (use Repository Layer)
- PII logging

**Pattern:**
```typescript
Request → Validate (Zod) → Authorize → Service.method() → Format Response
```

### 3.3 Server Actions Layer (`/src/lib/actions`)

**Purpose:** Form submissions and mutations from Client Components

**What belongs here:**
- Form handling
- Input validation
- Calling service methods
- Cache revalidation (revalidatePath, revalidateTag)

**What does NOT belong here:**
- Complex business logic (delegate to Service Layer)
- Direct database operations

### 3.4 Service Layer (`/src/lib/services`)

**Purpose:** Business logic, domain rules, orchestration

**What belongs here:**
- Business rules and validations
- Multi-step operations
- Transaction coordination
- PII encryption/decryption logic
- External API integration orchestration

**What does NOT belong here:**
- HTTP concerns (status codes, headers)
- UI concerns
- Direct SQL queries (use Repository Layer)

### 3.5 Repository Layer (Data Access)

**Purpose:** Database operations, query construction

**What belongs here:**
- Database queries (Prisma, Drizzle, or raw SQL)
- Query optimization
- Soft delete logic
- Field selection (never SELECT *)

**What does NOT belong here:**
- Business logic
- HTTP concerns
- External API calls

---

## 4.0 Data Flow Patterns

### 4.1 Read Operations (Server Components)

```
Page Request
    │
    ▼
Server Component (RSC)
    │
    ├─► Direct fetch in RSC (preferred)
    │   └─► Service Layer (if complex logic needed)
    │       └─► Repository Layer
    │           └─► Database
    │
    ▼
Render HTML (streamed to client)
```

### 4.2 Write Operations (Server Actions)

```
Form Submit / Button Click
    │
    ▼
Client Component (with 'use client')
    │
    ├─► Server Action (in /lib/actions/)
    │   └─► Validate Input (Zod)
    │       └─► Service Layer
    │           └─► Repository Layer
    │               └─► Database
    │                   └─► Return Result
    │
    ├─► revalidatePath() / revalidateTag()
    │
    ▼
UI Update (React handles reconciliation)
```

### 4.3 API Requests (External/AJAX)

```
HTTP Request
    │
    ▼
Route Handler (/app/(api)/...)
    │
    ├─► Parse & Validate Request (Zod)
    │   └─► Authorize (check session/token)
    │       └─► Service Layer
    │           └─► Repository Layer
    │               └─► Database
    │
    ▼
HTTP Response (JSON)
```

---

## 5.0 Vercel Deployment Architecture

### 5.1 Deployment Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        GIT REPOSITORY                            │
│                                                                  │
│  main branch ──────────────────► Production Environment         │
│  feature/* branches ───────────► Preview Environments           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      VERCEL BUILD PIPELINE                       │
│                                                                  │
│  1. Install dependencies (npm ci)                               │
│  2. Run linting (eslint)                                        │
│  3. Run type checking (tsc --noEmit)                           │
│  4. Run tests (jest, playwright)                                │
│  5. Build application (next build)                              │
│  6. Deploy to edge network                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    VERCEL EDGE NETWORK                           │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Static    │  │  Serverless │  │    Edge     │             │
│  │   Assets    │  │  Functions  │  │  Functions  │             │
│  │   (CDN)     │  │ (API/Pages) │  │ (Middleware)│             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Environment Strategy

| Environment | Branch | Purpose | Data |
|-------------|--------|---------|------|
| **Production** | `main` | Live users | Real data |
| **Preview** | `feature/*` | Testing, QA | Test/synthetic data |
| **Development** | local | Development | Local/mock data |

### 5.3 Environment Variables by Scope

```
Production Environment (Vercel Dashboard)
├── DATABASE_URL          → Production PostgreSQL
├── NEXTAUTH_SECRET       → Production secret (32+ chars)
└── NEXT_PUBLIC_APP_URL   → https://your-domain.com

Preview Environment (Vercel Dashboard)
├── DATABASE_URL          → Staging PostgreSQL
├── NEXTAUTH_SECRET       → Preview secret
└── NEXT_PUBLIC_APP_URL   → Preview URL

Local Development (.env.local - gitignored)
├── Only non-sensitive overrides
└── Pull secrets with: vercel env pull
```

### 5.4 Function Types & Limits

| Function Type | Use Case | Timeout | Size Limit |
|---------------|----------|---------|------------|
| **Serverless** | API routes, SSR pages | 10s (Hobby), 60s (Pro) | 50MB |
| **Edge** | Middleware, fast APIs | 30s | 1MB |
| **Static** | Pre-rendered pages | N/A | N/A |

---

## 6.0 Key Architectural Decisions

### 6.1 Server Components First

**Decision:** All components are React Server Components by default.

**Rationale:**
- Reduced client bundle size
- Direct database access without API layer
- Better SEO and initial page load
- Secure: secrets never reach client

**Implication:** Only use `'use client'` when requiring state, effects, or browser APIs.

### 6.2 Soft Deletes Only

**Decision:** No hard deletes. All deletions are soft (set `deletedAt` timestamp).

**Rationale:**
- Audit trail preservation
- GDPR compliance (can prove what data existed)
- Easy recovery from accidental deletions
- Referential integrity maintained

**Implementation:**
```typescript
// Repository pattern
async delete(id: string): Promise<void> {
  await db.update(table).set({ deletedAt: new Date() }).where(eq(table.id, id));
}

// All queries filter deleted records
const activeRecords = await db.select().from(table).where(isNull(table.deletedAt));
```

### 6.3 Monetary Values as Integers

**Decision:** Store all monetary values as integers (cents/pence), never floats.

**Rationale:**
- Floating-point arithmetic errors avoided
- Precise calculations for financial data
- Consistent across all services

**Implementation:**
```typescript
// Store
const priceInCents = Math.round(priceInDollars * 100);

// Display
const displayPrice = (cents: number) => (cents / 100).toFixed(2);
```

### 6.4 PII Encryption at Rest

**Decision:** All PII is encrypted in the database and decrypted only in the Service Layer.

**Rationale:**
- Defense in depth
- Database breach doesn't expose raw PII
- Centralized decryption logic for auditing

**Implementation:**
```typescript
// Service Layer only
const user = await userRepository.findById(id);
const decryptedEmail = decrypt(user.encryptedEmail);
```

### 6.5 Validation at Boundaries

**Decision:** All external input is validated with Zod at the application boundary.

**Rationale:**
- Type safety from edge to core
- Fail fast on invalid input
- Single source of truth for validation rules

**Boundaries:**
- API Route Handlers (request body)
- Server Actions (form data)
- Environment variables (startup)
- External API responses

---

## 7.0 External Dependencies

### 7.1 Required Services

| Service | Purpose | Critical? | Fallback |
|---------|---------|-----------|----------|
| **PostgreSQL** | Primary datastore | Yes | None |
| **Redis** | Caching, rate limiting | No | Graceful degradation |
| **Auth Provider** | Authentication | Yes | None |

### 7.2 Integration Patterns

**Authentication:**
- Session managed via NextAuth.js
- Tokens stored in HttpOnly cookies
- Middleware validates on every request

**External APIs:**
- NEVER log tokens, secrets, or sensitive data
- Use idempotency keys for mutations when available
- Implement proper error handling and retries

---

## 8.0 Security Architecture

### 8.1 Defense Layers

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1: Edge (Vercel)                                          │
│ - DDoS protection                                               │
│ - SSL/TLS termination                                           │
│ - Geographic restrictions (if configured)                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 2: Middleware (/src/middleware.ts)                        │
│ - Authentication check                                          │
│ - Rate limiting                                                 │
│ - CSRF protection                                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 3: Application                                            │
│ - Input validation (Zod)                                        │
│ - Authorization checks                                          │
│ - Output encoding                                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 4: Data                                                   │
│ - PII encryption                                                │
│ - Parameterized queries                                         │
│ - Least privilege access                                        │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 PII Handling Rules

| Data Type | Storage | Logging | Display |
|-----------|---------|---------|---------|
| Email | Encrypted | Never | Masked (j***@example.com) |
| Password | Hashed (bcrypt) | Never | Never |
| Phone | Encrypted | Never | Masked (***-***-1234) |
| IP Address | Hashed or omitted | Never raw | Never |
| Names | Encrypted | Never | As needed |
| API Keys/Tokens | Encrypted | Never | Never |

---

## 9.0 File Structure Reference

```
/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (api)/             # API route handlers
│   │   │   └── [resource]/
│   │   │       └── route.ts
│   │   ├── (features)/        # Feature routes
│   │   │   ├── auth/
│   │   │   ├── dashboard/
│   │   │   └── settings/
│   │   ├── layout.tsx         # Root layout
│   │   └── page.tsx           # Home page
│   │
│   ├── components/
│   │   ├── ui/                # Primitives (Button, Input)
│   │   ├── layout/            # Structure (Header, Footer)
│   │   └── features/          # Feature-specific
│   │
│   ├── lib/
│   │   ├── actions/           # Server Actions
│   │   ├── services/          # Business logic
│   │   ├── schemas/           # Zod schemas
│   │   ├── stores/            # Zustand stores
│   │   ├── utils/             # Pure utilities
│   │   └── env.ts             # Environment validation
│   │
│   ├── hooks/                 # Custom React hooks
│   │
│   └── types/                 # TypeScript types
│
├── tests/                     # Unit & integration tests
├── e2e/                       # Playwright E2E tests
├── public/                    # Static assets
│
├── .env.local                 # Local env (gitignored)
├── next.config.js             # Next.js configuration
├── tailwind.config.ts         # Tailwind configuration
├── vercel.json                # Vercel configuration
│
├── AGENTS.md                  # Project rules
├── ARCHITECTURE.md            # This file
└── .claude/
    └── skills/                # Claude Code skills
```

---

## 10.0 Decision Log

| Date | Decision | Rationale | Impact |
|------|----------|-----------|--------|
| Initial | Server Components default | Performance, security | All components RSC unless needed |
| Initial | Vercel-only deployment | Consistency, integration | No self-hosting option |
| Initial | Zustand for global state | Simplicity, bundle size | No Redux |
| Initial | Soft deletes only | Audit, compliance | Extra column in all tables |
| Initial | Integers for money | Precision | Convert on display |

---

## 11.0 Anti-Patterns to Avoid

### ❌ Direct Database Access in Components

```typescript
// WRONG
export async function UserList() {
  const users = await db.query('SELECT * FROM users');
  return <ul>{/* ... */}</ul>;
}
```

### ✅ Use Service/Repository Layers

```typescript
// CORRECT
export async function UserList() {
  const users = await userService.findAll();
  return <ul>{/* ... */}</ul>;
}
```

### ❌ Business Logic in Route Handlers

```typescript
// WRONG
export async function POST(request: NextRequest) {
  const body = await request.json();
  // 50 lines of business logic here
}
```

### ✅ Delegate to Service Layer

```typescript
// CORRECT
export async function POST(request: NextRequest) {
  const body = await request.json();
  const validated = CreateUserSchema.parse(body);
  const result = await userService.create(validated);
  return NextResponse.json(result, { status: 201 });
}
```

### ❌ PII in Logs

```typescript
// WRONG
console.log('User created:', { email: user.email, ip: request.ip });
```

### ✅ Safe Identifiers Only

```typescript
// CORRECT
console.log('User created:', { userId: user.id, timestamp: Date.now() });
```

---

## 12.0 References

- **AGENTS.md**: Project rules and coding standards
- **`.claude/skills/`**: Detailed implementation patterns
- **Next.js Docs**: https://nextjs.org/docs
- **Vercel Docs**: https://vercel.com/docs
