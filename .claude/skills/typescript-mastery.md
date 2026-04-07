# TypeScript Mastery

> Skill for advanced TypeScript patterns required by AGENTS.md standards

---

## Branded Types for Nominal Typing

AGENTS.md §3.1.4 requires branded types for business primitives. Use this pattern:

```typescript
// Define branded type
type Brand<T, B> = T & { readonly __brand: B };

// Business primitives
type UserId = Brand<string, 'UserId'>;
type ProductId = Brand<string, 'ProductId'>;
type Email = Brand<string, 'Email'>;

// Constructor functions with validation
const createUserId = (id: string): UserId => {
  if (!id || id.length < 8) {
    throw new Error('Invalid UserId format');
  }
  return id as UserId;
};

const createEmail = (email: string): Email => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    throw new Error('Invalid email format');
  }
  return email as Email;
};
```

**Why**: Prevents logical errors like passing a `ProductId` where a `UserId` is expected.

---

## Strict Type Annotations at Boundaries

AGENTS.md §3.1.3 requires explicit type annotations for function signatures and API responses.

```typescript
// ✅ GOOD: Explicit return type
export function calculateDiscount(price: number, percentage: number): number {
  return price * (1 - percentage / 100);
}

// ✅ GOOD: Explicit API response type
type ApiResponse<T> = {
  data: T;
  meta: {
    timestamp: string;
    requestId: string;
  };
};

export async function fetchUser(id: UserId): Promise<ApiResponse<User>> {
  // Implementation
}

// ❌ BAD: Implicit return type
export function calculateDiscount(price: number, percentage: number) {
  return price * (1 - percentage / 100);
}
```

---

## Replacing `any` with `unknown`

AGENTS.md §3.1.2 prohibits the `any` type. Use `unknown` with type guards:

```typescript
// ❌ BAD: Using any
function processData(data: any) {
  return data.value;
}

// ✅ GOOD: Using unknown with type guard
function isValidPayload(data: unknown): data is { value: string } {
  return (
    typeof data === 'object' &&
    data !== null &&
    'value' in data &&
    typeof (data as { value: unknown }).value === 'string'
  );
}

function processData(data: unknown): string {
  if (isValidPayload(data)) {
    return data.value; // TypeScript knows data.value is string
  }
  throw new Error('Invalid payload structure');
}
```

---

## Discriminated Unions for State Modeling

Use discriminated unions for type-safe state handling:

```typescript
// API request states
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Type-safe state handling
function renderUserState(state: RequestState<User>): string {
  switch (state.status) {
    case 'idle':
      return 'Ready to load';
    case 'loading':
      return 'Loading...';
    case 'success':
      return `Welcome, ${state.data.name}`; // TypeScript knows data exists
    case 'error':
      return `Error: ${state.error.message}`; // TypeScript knows error exists
  }
}
```

---

## Utility Types for API Responses

```typescript
// Make specific fields required
type WithRequired<T, K extends keyof T> = T & { [P in K]-?: T[P] };

// Make specific fields optional
type WithOptional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// Deep partial for nested objects
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Non-nullable fields
type NonNullableFields<T> = {
  [P in keyof T]: NonNullable<T[P]>;
};

// Usage
type UserUpdate = WithOptional<User, 'createdAt' | 'id'>;
type UserPatch = DeepPartial<User>;
```

---

## Generic Constraints for Type Safety

```typescript
// Constrained generic for database entities
type EntityBase = {
  id: string;
  createdAt: Date;
  updatedAt: Date;
};

async function findById<T extends EntityBase>(
  repository: Repository<T>,
  id: string
): Promise<T | null> {
  return repository.findOne({ where: { id } });
}

// Constrained generic for API handlers
type ApiHandler<TInput, TOutput> = (input: TInput) => Promise<TOutput>;

function withValidation<TInput, TOutput>(
  schema: ZodSchema<TInput>,
  handler: ApiHandler<TInput, TOutput>
): ApiHandler<unknown, TOutput> {
  return async (input: unknown) => {
    const validated = schema.parse(input);
    return handler(validated);
  };
}
```

---

## Const Assertions for Literal Types

```typescript
// HTTP methods as literal types
const HTTP_METHODS = ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'] as const;
type HttpMethod = (typeof HTTP_METHODS)[number];

// Route configuration
const ROUTES = {
  users: '/api/users',
  products: '/api/products',
  orders: '/api/orders',
} as const;

type RouteKey = keyof typeof ROUTES;
type RoutePath = (typeof ROUTES)[RouteKey];

// Status codes
const STATUS_CODES = {
  OK: 200,
  CREATED: 201,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  NOT_FOUND: 404,
  INTERNAL_ERROR: 500,
} as const;

type StatusCode = (typeof STATUS_CODES)[keyof typeof STATUS_CODES];
```

---

## Template Literal Types

```typescript
// API endpoint builder
type ApiVersion = 'v1' | 'v2';
type Resource = 'users' | 'products' | 'orders';
type ApiEndpoint = `/api/${ApiVersion}/${Resource}`;

// Event naming
type DomainEvent = 'user' | 'order' | 'product';
type EventAction = 'created' | 'updated' | 'deleted';
type EventName = `${DomainEvent}.${EventAction}`;

// CSS class builder
type Size = 'sm' | 'md' | 'lg' | 'xl';
type Variant = 'primary' | 'secondary' | 'danger';
type ButtonClass = `btn-${Size}-${Variant}`;
```

---

## Inference Helpers

```typescript
// Infer return type of async function
type AsyncReturnType<T extends (...args: unknown[]) => Promise<unknown>> =
  T extends (...args: unknown[]) => Promise<infer R> ? R : never;

// Infer props from component
type ComponentProps<T> = T extends React.ComponentType<infer P> ? P : never;

// Infer array element type
type ArrayElement<T> = T extends (infer E)[] ? E : never;

// Usage
async function getUser(): Promise<User> {
  // ...
}
type UserType = AsyncReturnType<typeof getUser>; // User
```

---

## Strict tsconfig.json

AGENTS.md §3.1.1 mandates strict mode. Required configuration:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

---

## Anti-Patterns to Avoid

```typescript
// ❌ Using any
const data: any = fetchData();

// ❌ Type assertion without validation
const user = data as User;

// ❌ Non-null assertion without checking
const name = user!.name;

// ❌ Implicit any in callbacks
items.map((item) => item.value); // if item type not inferred

// ❌ Default exports (AGENTS.md §3.2.4)
export default function handler() {}

// ✅ Named exports only
export function handler() {}
export { handler };
```
