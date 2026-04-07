---
paths:
  - "src/**/*.ts"
  - "src/**/*.tsx"
---

# Debugging Skills

> Systematic approaches to debugging Next.js TypeScript applications

---

## Debugging Philosophy

Per AGENTS.md guidelines:
1. **Address root causes**, not symptoms
2. **Add descriptive logging** to track state
3. **Add test functions** to isolate problems
4. Only make code changes when **certain** of the solution

---

## Browser DevTools Mastery

### React DevTools

```tsx
// Install React DevTools browser extension
// Access via: F12 → Components tab

// Debug component props and state
// 1. Select component in tree
// 2. View props, state, hooks in right panel
// 3. Edit state directly for testing

// Finding component source
// Click "< >" icon to jump to source in Sources tab
```

### Network Tab Analysis

```typescript
// Debugging API calls:
// 1. Filter by "Fetch/XHR"
// 2. Click request to see:
//    - Headers (check Authorization, Content-Type)
//    - Payload (request body)
//    - Response (actual data returned)
//    - Timing (identify slow requests)

// Common issues to check:
// - Status codes (400 = bad request, 401 = auth, 500 = server error)
// - CORS errors (check Origin header)
// - Missing headers
// - Malformed request body
```

### Console Patterns

```typescript
// Structured logging for debugging
console.log('%c[Component]', 'color: blue', 'State update:', state);
console.log('%c[API]', 'color: green', 'Request:', { endpoint, params });
console.log('%c[ERROR]', 'color: red', 'Failed:', error.message);

// Group related logs
console.group('Form Submission');
console.log('Form data:', formData);
console.log('Validation result:', validationResult);
console.log('API response:', response);
console.groupEnd();

// Table for array/object data
console.table(users);

// Timing operations
console.time('dataFetch');
const data = await fetchData();
console.timeEnd('dataFetch'); // Outputs: dataFetch: 123.45ms

// Stack trace
console.trace('How did we get here?');
```

---

## Server-Side Debugging

### Next.js Server Component Debugging

```typescript
// Server Components run on the server - use console.log
// Output appears in terminal, NOT browser console

export async function UserList() {
  console.log('[UserList] Fetching users...');
  
  const users = await fetchUsers();
  console.log('[UserList] Fetched:', users.length, 'users');
  
  return <ul>{/* ... */}</ul>;
}
```

### API Route Debugging

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest): Promise<NextResponse> {
  // Log request details (without PII)
  console.log('[POST /api/users]', {
    method: request.method,
    contentType: request.headers.get('content-type'),
    timestamp: new Date().toISOString(),
  });

  try {
    const body = await request.json();
    console.log('[POST /api/users] Body keys:', Object.keys(body));
    
    // ... processing
    
    console.log('[POST /api/users] Success');
    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('[POST /api/users] Error:', {
      name: error instanceof Error ? error.name : 'Unknown',
      message: error instanceof Error ? error.message : 'Unknown error',
    });
    
    return NextResponse.json({ error: 'Failed' }, { status: 500 });
  }
}
```

### VS Code Debugging Setup

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js: debug server-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev"
    },
    {
      "name": "Next.js: debug client-side",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000"
    },
    {
      "name": "Next.js: debug full stack",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev",
      "serverReadyAction": {
        "pattern": "- Local:.+(https?://.+)",
        "uriFormat": "%s",
        "action": "debugWithChrome"
      }
    }
  ]
}
```

---

## Systematic Debugging Approach

### 1. Problem Isolation

```typescript
// Step 1: Identify the smallest reproducible case
// Can you reproduce with:
// - Fresh browser session?
// - Different user?
// - Different data?

// Step 2: Add logging at boundaries
export async function processOrder(orderId: string) {
  console.log('[processOrder] START', { orderId });
  
  const order = await fetchOrder(orderId);
  console.log('[processOrder] Fetched order', { found: !!order });
  
  const validated = validateOrder(order);
  console.log('[processOrder] Validation', { valid: validated.success });
  
  const result = await submitOrder(order);
  console.log('[processOrder] Submit result', { success: result.success });
  
  console.log('[processOrder] END');
  return result;
}
```

### 2. Binary Search Debugging

```typescript
// When bug is in a large function, use binary search:
// 1. Add log in middle of function
// 2. Determine if bug is before or after
// 3. Add log in middle of remaining section
// 4. Repeat until narrowed down

function complexProcess(data: InputData): Result {
  console.log('[DEBUG 1] Input:', data);
  
  const step1 = transformA(data);
  console.log('[DEBUG 2] After transformA');
  
  const step2 = transformB(step1);
  console.log('[DEBUG 3] After transformB'); // Bug happens here!
  
  const step3 = transformC(step2);
  console.log('[DEBUG 4] After transformC');
  
  return finalize(step3);
}
```

### 3. Hypothesis-Driven Debugging

```typescript
// Document your hypotheses
/*
 * BUG: Order total is incorrect
 * 
 * Hypothesis 1: Discount not being applied
 * Test: Log discount value before and after application
 * Result: Discount is correct
 * 
 * Hypothesis 2: Tax calculation is wrong
 * Test: Log tax rate and calculated tax
 * Result: CONFIRMED - tax rate is undefined, defaults to 0
 * 
 * Root Cause: Tax rate not loaded from config
 * Fix: Ensure config is loaded before calculation
 */
```

---

## Common Bug Patterns

### Async/Await Issues

```typescript
// ❌ BUG: Forgot await
async function fetchAndProcess() {
  const data = fetchData(); // Missing await!
  console.log(data); // Logs Promise, not data
}

// ❌ BUG: Array methods don't await
async function processAll(items: Item[]) {
  items.forEach(async (item) => {
    await processItem(item); // This doesn't block!
  });
  console.log('Done'); // Runs before processing finishes
}

// ✅ FIX: Use Promise.all or for...of
async function processAll(items: Item[]) {
  await Promise.all(items.map((item) => processItem(item)));
  // Or for sequential processing:
  for (const item of items) {
    await processItem(item);
  }
  console.log('Done');
}
```

### State Update Issues

```typescript
// ❌ BUG: State not updating
function Counter() {
  const [count, setCount] = useState(0);
  
  const incrementTwice = () => {
    setCount(count + 1); // Uses stale count
    setCount(count + 1); // Still uses same stale count
  };
}

// ✅ FIX: Use functional update
function Counter() {
  const [count, setCount] = useState(0);
  
  const incrementTwice = () => {
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
  };
}
```

### Closure Issues

```typescript
// ❌ BUG: Stale closure
function SearchComponent() {
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log('Searching for:', query); // Stale query!
    }, 500);
    return () => clearTimeout(timer);
  }, []); // Missing query dependency
}

// ✅ FIX: Include dependency
function SearchComponent() {
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log('Searching for:', query);
    }, 500);
    return () => clearTimeout(timer);
  }, [query]); // Query in dependencies
}
```

### Type Coercion Issues

```typescript
// ❌ BUG: String comparison
const page = searchParams.get('page'); // Returns string
if (page === 1) { } // Always false! '1' !== 1

// ✅ FIX: Parse or use Zod
const pageSchema = z.coerce.number().int().positive().default(1);
const page = pageSchema.parse(searchParams.get('page'));
if (page === 1) { } // Works correctly
```

---

## Database Debugging

### Query Logging

```typescript
// Enable Prisma query logging
// prisma/schema.prisma or connection config
const prisma = new PrismaClient({
  log: [
    { emit: 'stdout', level: 'query' },
    { emit: 'stdout', level: 'error' },
    { emit: 'stdout', level: 'info' },
    { emit: 'stdout', level: 'warn' },
  ],
});

// Or for specific queries
const users = await prisma.user.findMany({
  where: { active: true },
});
console.log('[DB Query]', prisma.$queryRawUnsafe.toString());
```

### Connection Issues

```typescript
// Check database connectivity
async function checkDatabase(): Promise<boolean> {
  try {
    await prisma.$queryRaw`SELECT 1`;
    console.log('[DB] Connection successful');
    return true;
  } catch (error) {
    console.error('[DB] Connection failed:', {
      message: error instanceof Error ? error.message : 'Unknown',
      // Check: DATABASE_URL set? Network accessible? Credentials correct?
    });
    return false;
  }
}
```

---

## Memory Leak Detection

```typescript
// Common causes:
// 1. Event listeners not cleaned up
// 2. Intervals/timeouts not cleared
// 3. Subscriptions not unsubscribed
// 4. Large objects held in state

// ✅ Proper cleanup pattern
useEffect(() => {
  const handler = (e: Event) => { /* ... */ };
  window.addEventListener('resize', handler);
  
  const interval = setInterval(() => { /* ... */ }, 1000);
  
  const subscription = observable.subscribe(/* ... */);
  
  // Cleanup function - CRITICAL
  return () => {
    window.removeEventListener('resize', handler);
    clearInterval(interval);
    subscription.unsubscribe();
  };
}, []);
```

---

## Performance Debugging

### React Profiler

```tsx
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  console.log('[Profiler]', {
    id,
    phase,
    actualDuration: `${actualDuration.toFixed(2)}ms`,
  });
}

function App() {
  return (
    <Profiler id="Dashboard" onRender={onRenderCallback}>
      <Dashboard />
    </Profiler>
  );
}
```

### Identifying Re-renders

```typescript
// Track why component re-rendered
import { useRef, useEffect } from 'react';

function useWhyDidYouUpdate(name: string, props: Record<string, unknown>) {
  const previousProps = useRef<Record<string, unknown>>();

  useEffect(() => {
    if (previousProps.current) {
      const changedProps: Record<string, { from: unknown; to: unknown }> = {};
      
      Object.keys({ ...previousProps.current, ...props }).forEach((key) => {
        if (previousProps.current![key] !== props[key]) {
          changedProps[key] = {
            from: previousProps.current![key],
            to: props[key],
          };
        }
      });

      if (Object.keys(changedProps).length > 0) {
        console.log('[WhyDidYouUpdate]', name, changedProps);
      }
    }
    
    previousProps.current = props;
  });
}

// Usage
function MyComponent(props: Props) {
  useWhyDidYouUpdate('MyComponent', props);
  // ...
}
```

---

## Debugging Checklist

Before asking for help:

- [ ] Can you reproduce the bug consistently?
- [ ] Have you checked browser console for errors?
- [ ] Have you checked terminal for server errors?
- [ ] Have you verified the request/response in Network tab?
- [ ] Have you added logging to narrow down the issue?
- [ ] Have you checked if it works with different data?
- [ ] Have you searched for similar issues in the codebase?
- [ ] Have you formed a hypothesis about the root cause?
- [ ] Can you create a minimal reproduction?
