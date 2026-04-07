---
paths:
  - "src/app/**/*.tsx"
  - "src/components/**/*.tsx"
---

# Server Components Patterns

## When This Skill Applies

Use these patterns when building React Server Components (RSCs) in Next.js App Router.

## Core Principles

- **Server-first by default**: All components are RSCs unless explicitly marked with `'use client'`
- **No client hooks in RSCs**: useState, useEffect, useReducer are forbidden in server components
- **Data fetching in RSCs**: Fetch data directly in async components, not in client components

## Async Component Pattern

```tsx
// ✅ CORRECT: Async RSC with direct data fetching
export async function UserProfile({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## Parallel Data Fetching

```tsx
// ✅ CORRECT: Parallel fetches for independent data
export async function Dashboard({ userId }: { userId: string }) {
  const [user, posts, notifications] = await Promise.all([
    fetchUser(userId),
    fetchUserPosts(userId),
    fetchNotifications(userId),
  ]);

  return (
    <div>
      <UserHeader user={user} />
      <PostList posts={posts} />
      <NotificationBell count={notifications.length} />
    </div>
  );
}
```

## Sequential Data Fetching (When Necessary)

```tsx
// ✅ CORRECT: Sequential when data depends on previous result
export async function UserOrders({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  const orders = await fetchOrdersByRegion(user.regionId);

  return <OrderList orders={orders} />;
}
```

## Streaming with Suspense

```tsx
// ✅ CORRECT: Streaming slow components
import { Suspense } from 'react';

export function DashboardPage() {
  return (
    <div>
      <Header />
      <Suspense fallback={<AnalyticsSkeleton />}>
        <SlowAnalytics />
      </Suspense>
      <Suspense fallback={<RecommendationsSkeleton />}>
        <SlowRecommendations />
      </Suspense>
    </div>
  );
}
```

## Error Handling in RSCs

```tsx
// ✅ CORRECT: Error boundary integration
// app/dashboard/error.tsx
'use client';

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div role="alert">
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

## Passing Server Data to Client Components

```tsx
// ✅ CORRECT: Serialize data at the boundary
// Server Component
export async function ProductPage({ id }: { id: string }) {
  const product = await fetchProduct(id);

  return (
    <div>
      <ProductDetails product={product} />
      {/* Client component receives serializable data */}
      <AddToCartButton productId={product.id} price={product.price} />
    </div>
  );
}

// Client Component
'use client';

export function AddToCartButton({ 
  productId, 
  price 
}: { 
  productId: string; 
  price: number;
}) {
  const [isAdding, setIsAdding] = useState(false);
  // ... client-side logic
}
```

## Anti-Patterns to Avoid

```tsx
// ❌ WRONG: Using hooks in server components
export async function UserList() {
  const [users, setUsers] = useState([]); // ERROR: hooks not allowed
  useEffect(() => { /* ... */ }, []);      // ERROR: hooks not allowed
  
  return <div>{/* ... */}</div>;
}

// ❌ WRONG: Fetching in client components when RSC is possible
'use client';

export function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser); // Anti-pattern: use RSC instead
  }, [userId]);
  
  return <div>{/* ... */}</div>;
}

// ❌ WRONG: Passing non-serializable data to client components
export async function Page() {
  const db = await getDbConnection(); // Non-serializable!
  
  return <ClientComponent db={db} />; // ERROR: can't serialize
}
```

## Composition Pattern

```tsx
// ✅ CORRECT: Composing server and client components
// Server Component (layout)
export async function ProductLayout({ children }: { children: React.ReactNode }) {
  const categories = await fetchCategories();

  return (
    <div className="grid grid-cols-4">
      <aside>
        <CategoryNav categories={categories} />
      </aside>
      <main className="col-span-3">
        {children}
      </main>
    </div>
  );
}
```

## Cache and Revalidation

```tsx
// ✅ CORRECT: Using Next.js cache with revalidation
import { unstable_cache } from 'next/cache';

const getCachedUser = unstable_cache(
  async (userId: string) => fetchUser(userId),
  ['user'],
  { revalidate: 3600, tags: ['user'] }
);

export async function UserProfile({ userId }: { userId: string }) {
  const user = await getCachedUser(userId);
  return <Profile user={user} />;
}
```

## Checklist Before Creating a Component

1. Can this be a Server Component? (Default: YES)
2. Does it need state or effects? → Consider if state can be lifted or managed differently
3. Does it need browser APIs? → Only then use 'use client'
4. Is data fetching involved? → Do it in the RSC, not client
5. Are there slow operations? → Wrap in Suspense for streaming
