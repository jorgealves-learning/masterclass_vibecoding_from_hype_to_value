---
paths:
  - "src/app/**/*.ts"
  - "src/app/**/*.tsx"
  - "src/lib/actions/**/*.ts"
---

# Server Actions Skill

> Implementation patterns for Next.js Server Actions following AGENTS.md standards

---

## What Are Server Actions?

Server Actions are async functions that run on the server. They can be called from Client Components and are the primary way to handle form submissions and data mutations in the App Router.

## Declaring Server Actions

### In a Separate File (Recommended)

```typescript
// src/lib/actions/user.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

export type CreateUserInput = z.infer<typeof CreateUserSchema>;

export async function createUser(input: unknown): Promise<ActionResult<User>> {
  // Always validate input
  const parsed = CreateUserSchema.safeParse(input);
  
  if (!parsed.success) {
    return {
      success: false,
      error: 'Validation failed',
      details: parsed.error.flatten(),
    };
  }

  try {
    const user = await db.user.create({ data: parsed.data });
    
    // Revalidate relevant paths
    revalidatePath('/users');
    
    return { success: true, data: user };
  } catch (error) {
    console.error('createUser failed:', { timestamp: Date.now() });
    return { success: false, error: 'Failed to create user' };
  }
}
```

### Inline in Server Components

```tsx
// Only for simple, component-specific actions
export async function SettingsPage() {
  async function updateTheme(formData: FormData) {
    'use server';
    
    const theme = formData.get('theme');
    // Update theme...
  }

  return (
    <form action={updateTheme}>
      <select name="theme">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      <button type="submit">Save</button>
    </form>
  );
}
```

---

## Action Result Type Pattern

Define a consistent return type for all actions:

```typescript
// src/lib/actions/types.ts
export type ActionResult<T> =
  | { success: true; data: T }
  | { success: false; error: string; details?: unknown };

// Usage in action
export async function deleteUser(userId: string): Promise<ActionResult<void>> {
  try {
    await db.user.delete({ where: { id: userId } });
    revalidatePath('/users');
    return { success: true, data: undefined };
  } catch {
    return { success: false, error: 'Failed to delete user' };
  }
}
```

---

## Form Submission Pattern

### Basic Form Action

```tsx
// Client Component using server action
'use client';

import { useActionState } from 'react';
import { createUser } from '@/lib/actions/user';

export function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState: unknown, formData: FormData) => {
      const result = await createUser({
        email: formData.get('email'),
        name: formData.get('name'),
      });
      return result;
    },
    null
  );

  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <input name="name" type="text" required />
      
      {state?.success === false && (
        <p className="text-red-500">{state.error}</p>
      )}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

### With React Hook Form

```tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useTransition } from 'react';
import { createUser } from '@/lib/actions/user';

const schema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

type FormData = z.infer<typeof schema>;

export function CreateUserForm() {
  const [isPending, startTransition] = useTransition();
  const {
    register,
    handleSubmit,
    formState: { errors },
    setError,
    reset,
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => {
    startTransition(async () => {
      const result = await createUser(data);
      
      if (result.success) {
        reset();
      } else {
        setError('root', { message: result.error });
      }
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}
      
      {errors.root && <span>{errors.root.message}</span>}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

---

## Optimistic Updates

```tsx
'use client';

import { useOptimistic, useTransition } from 'react';
import { toggleLike } from '@/lib/actions/post';

type Post = {
  id: string;
  likes: number;
  isLiked: boolean;
};

export function LikeButton({ post }: { post: Post }) {
  const [isPending, startTransition] = useTransition();
  const [optimisticPost, setOptimisticPost] = useOptimistic(
    post,
    (state, newLiked: boolean) => ({
      ...state,
      isLiked: newLiked,
      likes: newLiked ? state.likes + 1 : state.likes - 1,
    })
  );

  const handleClick = () => {
    startTransition(async () => {
      setOptimisticPost(!optimisticPost.isLiked);
      await toggleLike(post.id);
    });
  };

  return (
    <button onClick={handleClick} disabled={isPending}>
      {optimisticPost.isLiked ? '❤️' : '🤍'} {optimisticPost.likes}
    </button>
  );
}
```

---

## Revalidation Strategies

### Path-Based Revalidation

```typescript
'use server';

import { revalidatePath } from 'next/cache';

export async function updatePost(postId: string, data: PostData) {
  await db.post.update({ where: { id: postId }, data });
  
  // Revalidate specific path
  revalidatePath(`/posts/${postId}`);
  
  // Revalidate layout and all nested segments
  revalidatePath('/posts', 'layout');
}
```

### Tag-Based Revalidation

```typescript
'use server';

import { revalidateTag } from 'next/cache';

export async function createComment(postId: string, content: string) {
  await db.comment.create({ data: { postId, content } });
  
  // Revalidate all fetches tagged with 'comments'
  revalidateTag('comments');
  revalidateTag(`post-${postId}`);
}

// In data fetching:
async function getComments(postId: string) {
  const res = await fetch(`/api/posts/${postId}/comments`, {
    next: { tags: ['comments', `post-${postId}`] },
  });
  return res.json();
}
```

---

## Progressive Enhancement

Server Actions work without JavaScript for basic forms:

```tsx
// This form works even if JS is disabled
export async function NewsletterSignup() {
  async function subscribe(formData: FormData) {
    'use server';
    
    const email = formData.get('email') as string;
    await addToNewsletter(email);
    redirect('/thank-you');
  }

  return (
    <form action={subscribe}>
      <input name="email" type="email" required />
      <button type="submit">Subscribe</button>
    </form>
  );
}
```

---

## Error Handling Best Practices

```typescript
'use server';

import { z } from 'zod';

const UpdateProfileSchema = z.object({
  name: z.string().min(2),
  bio: z.string().max(500).optional(),
});

export async function updateProfile(
  userId: string,
  input: unknown
): Promise<ActionResult<User>> {
  // 1. Validate input
  const parsed = UpdateProfileSchema.safeParse(input);
  if (!parsed.success) {
    return {
      success: false,
      error: 'Invalid input',
      details: parsed.error.flatten(),
    };
  }

  // 2. Authorization check
  const session = await getSession();
  if (!session || session.userId !== userId) {
    return { success: false, error: 'Unauthorized' };
  }

  // 3. Execute with error handling
  try {
    const user = await db.user.update({
      where: { id: userId },
      data: parsed.data,
    });
    
    revalidatePath(`/profile/${userId}`);
    return { success: true, data: user };
  } catch (error) {
    // Log without PII
    console.error('updateProfile failed:', {
      userId: userId.slice(0, 8) + '...',
      timestamp: Date.now(),
    });
    
    return { success: false, error: 'Failed to update profile' };
  }
}
```

---

## File Upload with Server Actions

```typescript
'use server';

import { z } from 'zod';

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];

export async function uploadAvatar(formData: FormData): Promise<ActionResult<string>> {
  const file = formData.get('avatar');
  
  if (!file || !(file instanceof File)) {
    return { success: false, error: 'No file provided' };
  }

  if (!ALLOWED_TYPES.includes(file.type)) {
    return { success: false, error: 'Invalid file type' };
  }

  if (file.size > MAX_FILE_SIZE) {
    return { success: false, error: 'File too large (max 5MB)' };
  }

  try {
    const bytes = await file.arrayBuffer();
    const buffer = Buffer.from(bytes);
    
    // Upload to storage service
    const url = await uploadToStorage(buffer, file.name);
    
    return { success: true, data: url };
  } catch {
    return { success: false, error: 'Upload failed' };
  }
}
```

---

## Anti-Patterns to Avoid

### ❌ Not Validating Input

```typescript
// BAD: Trusting client input
'use server';

export async function updateUser(data: UserData) {
  await db.user.update({ data }); // Dangerous!
}
```

### ❌ Exposing Sensitive Data in Errors

```typescript
// BAD: Leaking internal details
'use server';

export async function createOrder(data: OrderData) {
  try {
    return await db.order.create({ data });
  } catch (error) {
    // Exposes database schema and internal errors
    return { error: error.message };
  }
}
```

### ❌ Missing Authorization

```typescript
// BAD: No auth check
'use server';

export async function deleteUser(userId: string) {
  await db.user.delete({ where: { id: userId } });
}

// GOOD: With auth check
'use server';

export async function deleteUser(userId: string): Promise<ActionResult<void>> {
  const session = await getSession();
  
  if (!session?.user?.isAdmin) {
    return { success: false, error: 'Unauthorized' };
  }
  
  await db.user.delete({ where: { id: userId } });
  revalidatePath('/admin/users');
  return { success: true, data: undefined };
}
```

### ❌ Logging PII

```typescript
// BAD: Logging user data
console.log('User update:', { email: user.email, address: user.address });

// GOOD: Safe logging
console.log('User update:', { userId: user.id, timestamp: Date.now() });
```

---

## Testing Server Actions

```typescript
import { describe, it, expect, vi } from 'vitest';
import { createUser } from './user';

describe('createUser', () => {
  it('validates input', async () => {
    const result = await createUser({ email: 'invalid', name: '' });
    
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error).toBe('Validation failed');
    }
  });

  it('creates user with valid input', async () => {
    const result = await createUser({
      email: 'test@example.com',
      name: 'Test User',
    });
    
    expect(result.success).toBe(true);
    if (result.success) {
      expect(result.data.email).toBe('test@example.com');
    }
  });
});
```
