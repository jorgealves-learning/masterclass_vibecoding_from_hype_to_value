---
paths:
  - "src/**/*"
---

# Directory Structure Patterns

> Skill for organizing code according to AGENTS.md Section 2.1 Directory Structure Mandate

---

## Mandated Structure

```
project-root/
├── src/                          # All application source code (§2.1.1)
│   ├── app/                      # Next.js App Router
│   │   ├── (api)/               # API route handlers (§2.1.5)
│   │   │   ├── users/
│   │   │   │   └── route.ts
│   │   │   └── products/
│   │   │       └── route.ts
│   │   ├── (features)/          # Feature-based routing (§2.1.2)
│   │   │   ├── auth/
│   │   │   │   ├── login/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── register/
│   │   │   │       └── page.tsx
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/              # Component stratification (§2.1.3)
│   │   ├── ui/                  # Generic primitives
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   └── card.tsx
│   │   ├── layout/              # Page structure components
│   │   │   ├── header.tsx
│   │   │   ├── sidebar.tsx
│   │   │   └── footer.tsx
│   │   └── features/            # Feature-specific components
│   │       ├── auth/
│   │       │   ├── login-form.tsx
│   │       │   └── register-form.tsx
│   │       └── dashboard/
│   │           ├── stats-card.tsx
│   │           └── activity-feed.tsx
│   ├── lib/                     # Shared logic & utilities (§2.1.4)
│   │   ├── actions/             # Server actions
│   │   │   └── user.ts
│   │   ├── stores/              # Zustand stores
│   │   │   └── cart.ts
│   │   ├── services/            # Business logic services
│   │   │   └── user-service.ts
│   │   ├── utils/               # Pure utility functions
│   │   │   └── format.ts
│   │   ├── schemas/             # Zod schemas
│   │   │   └── user.ts
│   │   └── env.ts               # Environment validation
│   ├── hooks/                   # Custom React hooks (§2.1.4)
│   │   ├── use-debounce.ts
│   │   └── use-local-storage.ts
│   └── types/                   # Shared TypeScript types
│       └── index.ts
├── tests/                       # Test files
│   ├── unit/
│   └── integration/
├── e2e/                         # Playwright E2E tests
│   └── auth.spec.ts
├── public/                      # Static assets
└── .env.local                   # Local env overrides (gitignored)
```

---

## Route Groups Explained

Route groups use parentheses `()` to organize routes without affecting the URL path:

```
src/app/
├── (api)/              # Groups all API routes
│   └── users/
│       └── route.ts    # URL: /users (not /api/users)
├── (features)/         # Groups feature pages
│   └── dashboard/
│       └── page.tsx    # URL: /dashboard (not /features/dashboard)
└── (marketing)/        # Groups marketing pages
    ├── about/
    │   └── page.tsx    # URL: /about
    └── pricing/
        └── page.tsx    # URL: /pricing
```

---

## Component Stratification Rules

### `/ui` - Generic Primitives

Contains only unstyled or minimally styled building blocks:

```tsx
// src/components/ui/button.tsx
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        outline: 'border border-input hover:bg-accent',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants>;

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button className={buttonVariants({ variant, size, className })} {...props} />
  );
}
```

### `/layout` - Page Structure Components

Components responsible for overall page layout:

```tsx
// src/components/layout/header.tsx
import { Navigation } from './navigation';
import { UserMenu } from './user-menu';

export function Header() {
  return (
    <header className="sticky top-0 z-50 border-b bg-background">
      <div className="container flex h-16 items-center justify-between">
        <Navigation />
        <UserMenu />
      </div>
    </header>
  );
}
```

### `/features` - Feature-Specific Components

Components specific to a feature, composed of `/ui` and `/layout` components:

```tsx
// src/components/features/auth/login-form.tsx
'use client';

import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card } from '@/components/ui/card';

export function LoginForm() {
  return (
    <Card className="w-full max-w-md">
      <form>
        <Input type="email" placeholder="Email" />
        <Input type="password" placeholder="Password" />
        <Button type="submit">Sign In</Button>
      </form>
    </Card>
  );
}
```

---

## `/lib` Organization

### `/lib/actions` - Server Actions

```typescript
// src/lib/actions/user.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

export async function createUser(input: unknown) {
  const parsed = CreateUserSchema.safeParse(input);
  if (!parsed.success) {
    return { success: false, error: parsed.error.flatten() };
  }
  // ... create user
  revalidatePath('/users');
  return { success: true };
}
```

### `/lib/services` - Business Logic

```typescript
// src/lib/services/user-service.ts
import { prisma } from '@/lib/prisma';
import type { User, CreateUserInput } from '@/types';

export const userService = {
  async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { id } });
  },

  async create(data: CreateUserInput): Promise<User> {
    return prisma.user.create({ data });
  },

  async delete(id: string): Promise<void> {
    await prisma.user.delete({ where: { id } });
  },
};
```

### `/lib/schemas` - Zod Schemas

```typescript
// src/lib/schemas/user.ts
import { z } from 'zod';

export const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(2),
  createdAt: z.date(),
});

export const CreateUserSchema = UserSchema.omit({ id: true, createdAt: true });
export const UpdateUserSchema = CreateUserSchema.partial();

export type User = z.infer<typeof UserSchema>;
export type CreateUserInput = z.infer<typeof CreateUserSchema>;
export type UpdateUserInput = z.infer<typeof UpdateUserSchema>;
```

### `/lib/stores` - Zustand Stores

```typescript
// src/lib/stores/cart.ts
import { create } from 'zustand';

type CartItem = {
  productId: string;
  quantity: number;
};

type CartStore = {
  items: CartItem[];
  addItem: (productId: string) => void;
  removeItem: (productId: string) => void;
  clearCart: () => void;
};

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (productId) =>
    set((state) => ({
      items: [...state.items, { productId, quantity: 1 }],
    })),
  removeItem: (productId) =>
    set((state) => ({
      items: state.items.filter((item) => item.productId !== productId),
    })),
  clearCart: () => set({ items: [] }),
}));
```

---

## `/hooks` Organization

Custom hooks encapsulating reusable stateful logic:

```typescript
// src/hooks/use-debounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

```typescript
// src/hooks/use-local-storage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    if (typeof window !== 'undefined') {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue] as const;
}
```

---

## API Route Organization

All API routes must be in `/src/app/(api)/`:

```typescript
// src/app/(api)/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { userService } from '@/lib/services/user-service';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

export async function POST(request: NextRequest): Promise<NextResponse> {
  const body = await request.json();
  const parsed = CreateUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json(
      { error: 'Validation failed', details: parsed.error.flatten() },
      { status: 400 }
    );
  }

  const user = await userService.create(parsed.data);
  return NextResponse.json(user, { status: 201 });
}

export async function GET(): Promise<NextResponse> {
  const users = await userService.findAll();
  return NextResponse.json(users);
}
```

---

## Anti-Patterns to Avoid

### ❌ Code outside `/src`

```
// WRONG
/utils/helper.ts
/components/Button.tsx

// CORRECT
/src/lib/utils/helper.ts
/src/components/ui/button.tsx
```

### ❌ Direct database calls in components

```tsx
// WRONG: Database call in component
export async function UserList() {
  const users = await prisma.user.findMany(); // ❌
  return <ul>{/* ... */}</ul>;
}

// CORRECT: Use service layer
export async function UserList() {
  const users = await userService.findAll(); // ✅
  return <ul>{/* ... */}</ul>;
}
```

### ❌ Mixing component types in wrong directories

```
// WRONG
/src/components/ui/login-form.tsx     # Feature component in ui/
/src/components/features/button.tsx   # Primitive in features/

// CORRECT
/src/components/features/auth/login-form.tsx
/src/components/ui/button.tsx
```

### ❌ API routes outside (api) group

```
// WRONG
/src/app/users/api/route.ts

// CORRECT
/src/app/(api)/users/route.ts
```

---

## File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | kebab-case | `user-profile.tsx` |
| Hooks | kebab-case with `use-` prefix | `use-debounce.ts` |
| Utilities | kebab-case | `format-date.ts` |
| Types | kebab-case | `user-types.ts` |
| Schemas | kebab-case | `user-schema.ts` |
| Stores | kebab-case | `cart-store.ts` |
| Route files | Next.js convention | `page.tsx`, `route.ts`, `layout.tsx` |

---

## Import Aliases

Configure path aliases in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Usage:

```typescript
// Instead of relative imports
import { Button } from '../../../components/ui/button';

// Use aliases
import { Button } from '@/components/ui/button';
import { userService } from '@/lib/services/user-service';
import { useDebounce } from '@/hooks/use-debounce';
```
