---
paths:
  - "src/**/*.tsx"
  - "src/components/**/*"
---

# Client Components Skill

## When to Use Client Components

Use the `'use client'` directive ONLY when a component requires:

- **State**: `useState`, `useReducer`
- **Effects**: `useEffect`, `useLayoutEffect`
- **Browser APIs**: `window`, `document`, `localStorage`
- **Event Handlers**: `onClick`, `onChange`, `onSubmit`
- **Custom Hooks**: that use any of the above

## Required Pattern

Every client component MUST include a justification comment:

```tsx
// 'use client' justification: Requires useState for form input state and onClick handlers
'use client';

import { useState } from 'react';

export function SearchInput() {
  const [query, setQuery] = useState('');
  // ...
}
```

## Client Component Patterns

### State Management Pattern

```tsx
// 'use client' justification: Manages local form state with useState
'use client';

import { useState } from 'react';
import { z } from 'zod';

const formSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

export function ContactForm() {
  const [formData, setFormData] = useState({ email: '', name: '' });
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleChange = (field: string) => (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData(prev => ({ ...prev, [field]: e.target.value }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const result = formSchema.safeParse(formData);
    if (!result.success) {
      const fieldErrors: Record<string, string> = {};
      result.error.errors.forEach(err => {
        if (err.path[0]) fieldErrors[err.path[0].toString()] = err.message;
      });
      setErrors(fieldErrors);
      return;
    }
    // Submit validated data
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
    </form>
  );
}
```

### Effect Pattern with Cleanup

```tsx
// 'use client' justification: Requires useEffect for window resize listener
'use client';

import { useState, useEffect } from 'react';

export function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    // Set initial size
    handleResize();

    window.addEventListener('resize', handleResize);
    
    // Cleanup function - ALWAYS clean up listeners
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Empty deps = run once on mount

  return <span>{size.width} x {size.height}</span>;
}
```

### Zustand Store Integration

```tsx
// 'use client' justification: Subscribes to Zustand store for global state
'use client';

import { useCartStore } from '@/lib/stores/cart';

export function CartCount() {
  // Use selector to minimize re-renders
  const itemCount = useCartStore(state => state.items.length);

  return <span className="badge">{itemCount}</span>;
}
```

### React Hook Form Integration

```tsx
// 'use client' justification: Uses React Hook Form for controlled form state
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type FormData = z.infer<typeof schema>;

export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    // Handle submission
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
}
```

## Anti-Patterns to Avoid

### ❌ Client Component Without Justification

```tsx
'use client'; // NO! Missing justification comment

export function Button() {
  return <button>Click me</button>;
}
```

### ❌ Unnecessary Client Component

```tsx
// This should be a Server Component - no client features used
'use client';

export function UserCard({ user }: { user: User }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### ❌ Data Fetching in Client Component

```tsx
// 'use client' justification: [This is wrong - fetch should be in RSC]
'use client';

import { useEffect, useState } from 'react';

export function UserList() {
  const [users, setUsers] = useState([]);

  // ❌ WRONG: Data fetching should happen in Server Components
  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(setUsers);
  }, []);

  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

### ✅ Correct: Server Component with Client Island

```tsx
// ServerUserList.tsx (Server Component - no directive)
import { ClientUserActions } from './ClientUserActions';

async function getUsers() {
  const res = await fetch('https://api.example.com/users');
  return res.json();
}

export async function ServerUserList() {
  const users = await getUsers(); // Data fetching in RSC

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <ClientUserActions userId={user.id} /> {/* Client island for interactions */}
        </li>
      ))}
    </ul>
  );
}
```

```tsx
// ClientUserActions.tsx
// 'use client' justification: Requires onClick handlers for user actions
'use client';

export function ClientUserActions({ userId }: { userId: string }) {
  const handleDelete = () => {
    // Handle delete action
  };

  return (
    <button onClick={handleDelete}>Delete</button>
  );
}
```

## Composition Pattern: Server + Client

Keep client components as small "islands" within server components:

```tsx
// Page.tsx (Server Component)
import { Suspense } from 'react';
import { ServerDataDisplay } from '@/components/features/ServerDataDisplay';
import { ClientSearchInput } from '@/components/features/ClientSearchInput';

export default async function Page() {
  return (
    <div>
      <ClientSearchInput /> {/* Small client island */}
      <Suspense fallback={<Loading />}>
        <ServerDataDisplay /> {/* Server component with data */}
      </Suspense>
    </div>
  );
}
```

## Named Export Requirement

All components must use named exports (per AGENTS.md 3.2.4):

```tsx
// ✅ Correct
export function MyComponent() { }

// ❌ Wrong
export default function MyComponent() { }
```
