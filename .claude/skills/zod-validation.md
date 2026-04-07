---
paths:
  - "src/**/*.ts"
  - "src/**/*.tsx"
---

# Zod Schema Validation Skills

## Purpose

This skill covers runtime validation of external data using Zod, as mandated in AGENTS.md Section 3.1.5. All external data (API responses, form submissions, environment variables) must be validated at application boundaries.

## Core Patterns

### Environment Variable Validation

Create `/src/lib/env.ts` to validate environment variables at startup:

```typescript
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  // Add all required env vars here
});

export type Env = z.infer<typeof envSchema>;

function validateEnv(): Env {
  const parsed = envSchema.safeParse(process.env);
  
  if (!parsed.success) {
    console.error('❌ Invalid environment variables:', parsed.error.flatten().fieldErrors);
    throw new Error('Invalid environment variables');
  }
  
  return parsed.data;
}

export const env = validateEnv();
```

### API Request Validation in Route Handlers

```typescript
import { z } from 'zod';
import { NextRequest, NextResponse } from 'next/server';

const createUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  name: z.string().min(2, 'Name must be at least 2 characters'),
  age: z.number().int().positive().optional(),
});

type CreateUserInput = z.infer<typeof createUserSchema>;

export async function POST(request: NextRequest): Promise<NextResponse> {
  const body = await request.json();
  const parsed = createUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json(
      { error: 'Validation failed', details: parsed.error.flatten() },
      { status: 400 }
    );
  }

  const validatedData: CreateUserInput = parsed.data;
  // Process validated data...
}
```

### Form Validation with React Hook Form

```typescript
import { z } from 'zod';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const loginSchema = z.object({
  email: z.string().email('Please enter a valid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type LoginFormData = z.infer<typeof loginSchema>;

export function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = (data: LoginFormData) => {
    // data is fully typed and validated
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* form fields */}
    </form>
  );
}
```

### API Response Validation

```typescript
import { z } from 'zod';

const userResponseSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  createdAt: z.string().datetime(),
  profile: z.object({
    name: z.string(),
    avatar: z.string().url().nullable(),
  }).optional(),
});

type UserResponse = z.infer<typeof userResponseSchema>;

async function fetchUser(id: string): Promise<UserResponse> {
  const response = await fetch(`/api/users/${id}`);
  const data: unknown = await response.json();
  
  // Validate external data at the boundary
  return userResponseSchema.parse(data);
}
```

## Schema Composition Patterns

### Extending Schemas

```typescript
const baseUserSchema = z.object({
  email: z.string().email(),
  name: z.string(),
});

const createUserSchema = baseUserSchema.extend({
  password: z.string().min(8),
});

const updateUserSchema = baseUserSchema.partial();

const fullUserSchema = baseUserSchema.extend({
  id: z.string().uuid(),
  createdAt: z.date(),
});
```

### Discriminated Unions

```typescript
const notificationSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('email'),
    email: z.string().email(),
    subject: z.string(),
  }),
  z.object({
    type: z.literal('sms'),
    phoneNumber: z.string(),
    message: z.string().max(160),
  }),
  z.object({
    type: z.literal('push'),
    deviceToken: z.string(),
    title: z.string(),
    body: z.string(),
  }),
]);

type Notification = z.infer<typeof notificationSchema>;
```

### Array and Record Validation

```typescript
const tagsSchema = z.array(z.string()).min(1).max(10);

const configSchema = z.record(z.string(), z.unknown());

const paginatedSchema = <T extends z.ZodTypeAny>(itemSchema: T) =>
  z.object({
    items: z.array(itemSchema),
    total: z.number().int().nonnegative(),
    page: z.number().int().positive(),
    pageSize: z.number().int().positive(),
  });
```

## Branded Types with Zod

As per AGENTS.md Section 3.1.4, use branded types for domain-specific primitives:

```typescript
const UserIdSchema = z.string().uuid().brand<'UserId'>();
const ProductIdSchema = z.string().uuid().brand<'ProductId'>();
const EmailSchema = z.string().email().brand<'Email'>();

type UserId = z.infer<typeof UserIdSchema>;
type ProductId = z.infer<typeof ProductIdSchema>;
type Email = z.infer<typeof EmailSchema>;

// This prevents accidentally mixing up IDs
function getUser(userId: UserId): Promise<User> {
  // ...
}

// ❌ This will fail type checking
const productId = ProductIdSchema.parse('123e4567-e89b-12d3-a456-426614174000');
getUser(productId); // Type error!

// ✅ This works
const userId = UserIdSchema.parse('123e4567-e89b-12d3-a456-426614174000');
getUser(userId);
```

## Custom Error Messages

```typescript
const userSchema = z.object({
  email: z.string({
    required_error: 'Email is required',
    invalid_type_error: 'Email must be a string',
  }).email('Please provide a valid email address'),
  
  age: z.number({
    required_error: 'Age is required',
    invalid_type_error: 'Age must be a number',
  }).int('Age must be a whole number')
    .min(13, 'You must be at least 13 years old')
    .max(120, 'Age cannot exceed 120'),
});
```

## Transformations and Preprocessing

```typescript
const dateStringSchema = z.string().transform((str) => new Date(str));

const trimmedStringSchema = z.string().trim();

const slugSchema = z.string().transform((str) => 
  str.toLowerCase().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, '')
);

// Coercion for form data (strings to numbers/booleans)
const formSchema = z.object({
  quantity: z.coerce.number().int().positive(),
  isActive: z.coerce.boolean(),
  startDate: z.coerce.date(),
});
```

## Async Validation

```typescript
const uniqueEmailSchema = z.string().email().refine(
  async (email) => {
    const exists = await checkEmailExists(email);
    return !exists;
  },
  { message: 'Email already registered' }
);

// Use parseAsync for async refinements
const result = await uniqueEmailSchema.safeParseAsync(email);
```

## Anti-Patterns to Avoid

### ❌ Using `any` instead of `unknown`

```typescript
// Bad
const data: any = await response.json();
processUser(data); // No validation!

// Good
const data: unknown = await response.json();
const user = userSchema.parse(data); // Validated!
processUser(user);
```

### ❌ Skipping validation at boundaries

```typescript
// Bad - trusting external data
export async function POST(request: NextRequest) {
  const body = await request.json() as CreateUserInput; // Dangerous!
}

// Good - validating external data
export async function POST(request: NextRequest) {
  const body = await request.json();
  const parsed = createUserSchema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error }, { status: 400 });
  }
}
```

### ❌ Catching parse errors silently

```typescript
// Bad
try {
  const data = schema.parse(input);
} catch {
  // Silently failing
  return null;
}

// Good
const result = schema.safeParse(input);
if (!result.success) {
  console.error('Validation failed:', result.error.flatten());
  throw new ValidationError(result.error);
}
```

## Testing Schemas

```typescript
import { describe, it, expect } from '@jest/globals';

describe('userSchema', () => {
  it('accepts valid user data', () => {
    const validData = {
      email: 'user@example.com',
      name: 'John Doe',
    };
    
    expect(() => userSchema.parse(validData)).not.toThrow();
  });

  it('rejects invalid email', () => {
    const invalidData = {
      email: 'not-an-email',
      name: 'John Doe',
    };
    
    const result = userSchema.safeParse(invalidData);
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.flatten().fieldErrors.email).toBeDefined();
    }
  });

  it('provides helpful error messages', () => {
    const result = userSchema.safeParse({ email: '', name: '' });
    
    expect(result.success).toBe(false);
    if (!result.success) {
      const errors = result.error.flatten().fieldErrors;
      expect(errors.email?.[0]).toContain('email');
    }
  });
});
```
