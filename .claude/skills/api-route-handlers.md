---
paths:
  - "src/app/(api)/**/*.ts"
  - "src/app/api/**/*.ts"
---

# API Route Handler Implementation

This skill covers the implementation patterns for Next.js API route handlers following project standards.

## Route Handler Structure

Every route handler must follow this structure:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

// 1. Define request schema
const RequestSchema = z.object({
  // schema definition
});

// 2. Define response schema
const ResponseSchema = z.object({
  // schema definition
});

// 3. Type exports for consumers
export type RequestBody = z.infer<typeof RequestSchema>;
export type ResponseBody = z.infer<typeof ResponseSchema>;

// 4. Named export for HTTP method
export async function POST(request: NextRequest): Promise<NextResponse<ResponseBody>> {
  // implementation
}
```

## Request Validation Pattern

Always validate incoming data at the boundary:

```typescript
export async function POST(request: NextRequest): Promise<NextResponse> {
  // Parse JSON safely
  let body: unknown;
  try {
    body = await request.json();
  } catch {
    return NextResponse.json(
      { error: 'Invalid JSON body' },
      { status: 400 }
    );
  }

  // Validate with Zod
  const result = RequestSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json(
      { error: 'Validation failed', details: result.error.flatten() },
      { status: 400 }
    );
  }

  // Use validated data
  const validatedData = result.data;
  // ... continue processing
}
```

## Response Formatting Standards

### Success Responses

```typescript
// 200 OK - Resource retrieved
return NextResponse.json(data, { status: 200 });

// 201 Created - Resource created
return NextResponse.json(createdResource, { status: 201 });

// 204 No Content - Successful operation with no response body
return new NextResponse(null, { status: 204 });
```

### Error Responses

```typescript
// Standard error response shape
type ErrorResponse = {
  error: string;
  details?: unknown;
  code?: string;
};

// 400 Bad Request
return NextResponse.json(
  { error: 'Invalid request', code: 'INVALID_REQUEST' },
  { status: 400 }
);

// 401 Unauthorized
return NextResponse.json(
  { error: 'Authentication required', code: 'UNAUTHORIZED' },
  { status: 401 }
);

// 403 Forbidden
return NextResponse.json(
  { error: 'Insufficient permissions', code: 'FORBIDDEN' },
  { status: 403 }
);

// 404 Not Found
return NextResponse.json(
  { error: 'Resource not found', code: 'NOT_FOUND' },
  { status: 404 }
);

// 500 Internal Server Error
return NextResponse.json(
  { error: 'Internal server error', code: 'INTERNAL_ERROR' },
  { status: 500 }
);
```

## Error Handling Pattern

Wrap handlers with try-catch and log errors without PII:

```typescript
export async function GET(request: NextRequest): Promise<NextResponse> {
  try {
    // Handler logic
    const result = await fetchData();
    return NextResponse.json(result);
  } catch (error) {
    // Log error without PII
    console.error('GET /api/resource failed:', {
      errorMessage: error instanceof Error ? error.message : 'Unknown error',
      timestamp: new Date().toISOString(),
      // NEVER log: user emails, IPs, tokens, passwords
    });

    return NextResponse.json(
      { error: 'Internal server error', code: 'INTERNAL_ERROR' },
      { status: 500 }
    );
  }
}
```

## Query Parameter Handling

```typescript
export async function GET(request: NextRequest): Promise<NextResponse> {
  const searchParams = request.nextUrl.searchParams;
  
  // Define query schema
  const QuerySchema = z.object({
    page: z.coerce.number().int().positive().default(1),
    limit: z.coerce.number().int().min(1).max(100).default(20),
    search: z.string().optional(),
  });

  const result = QuerySchema.safeParse({
    page: searchParams.get('page'),
    limit: searchParams.get('limit'),
    search: searchParams.get('search'),
  });

  if (!result.success) {
    return NextResponse.json(
      { error: 'Invalid query parameters', details: result.error.flatten() },
      { status: 400 }
    );
  }

  const { page, limit, search } = result.data;
  // Use validated params
}
```

## Dynamic Route Parameters

For routes like `app/(api)/users/[id]/route.ts`:

```typescript
type RouteParams = {
  params: Promise<{ id: string }>;
};

export async function GET(
  request: NextRequest,
  { params }: RouteParams
): Promise<NextResponse> {
  const { id } = await params;
  
  // Validate ID format
  const IdSchema = z.string().uuid();
  const result = IdSchema.safeParse(id);
  
  if (!result.success) {
    return NextResponse.json(
      { error: 'Invalid ID format' },
      { status: 400 }
    );
  }

  // Fetch and return resource
}
```

## Headers and Authentication

```typescript
export async function GET(request: NextRequest): Promise<NextResponse> {
  // Get authorization header
  const authHeader = request.headers.get('authorization');
  
  if (!authHeader?.startsWith('Bearer ')) {
    return NextResponse.json(
      { error: 'Missing or invalid authorization header' },
      { status: 401 }
    );
  }

  const token = authHeader.slice(7);
  
  // Validate token (implement your auth logic)
  const user = await validateToken(token);
  
  if (!user) {
    return NextResponse.json(
      { error: 'Invalid token' },
      { status: 401 }
    );
  }

  // Continue with authenticated request
}
```

## Streaming Responses

For large data or real-time updates:

```typescript
export async function GET(request: NextRequest): Promise<Response> {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    async start(controller) {
      for await (const chunk of dataGenerator()) {
        controller.enqueue(encoder.encode(JSON.stringify(chunk) + '\n'));
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'application/x-ndjson',
      'Transfer-Encoding': 'chunked',
    },
  });
}
```

## File Upload Handling

```typescript
export async function POST(request: NextRequest): Promise<NextResponse> {
  const formData = await request.formData();
  const file = formData.get('file');

  if (!file || !(file instanceof File)) {
    return NextResponse.json(
      { error: 'No file provided' },
      { status: 400 }
    );
  }

  // Validate file type
  const allowedTypes = ['image/png', 'image/jpeg', 'application/pdf'];
  if (!allowedTypes.includes(file.type)) {
    return NextResponse.json(
      { error: 'Invalid file type' },
      { status: 400 }
    );
  }

  // Validate file size (e.g., 10MB max)
  const maxSize = 10 * 1024 * 1024;
  if (file.size > maxSize) {
    return NextResponse.json(
      { error: 'File too large' },
      { status: 400 }
    );
  }

  // Process file
  const bytes = await file.arrayBuffer();
  // ... save or process file
}
```

## Anti-Patterns to Avoid

### ❌ Direct database calls in route handlers

```typescript
// BAD: Direct database access
export async function GET() {
  const users = await db.query('SELECT * FROM users');
  return NextResponse.json(users);
}
```

### ✅ Use service layer abstraction

```typescript
// GOOD: Service layer abstraction
import { userService } from '@/lib/services/user';

export async function GET() {
  const users = await userService.findAll();
  return NextResponse.json(users);
}
```

### ❌ Logging PII

```typescript
// BAD: Logging sensitive data
console.log('User logged in:', { email: user.email, ip: request.ip });
```

### ✅ Log safe identifiers only

```typescript
// GOOD: Safe logging
console.log('User logged in:', { userId: user.id, timestamp: Date.now() });
```

### ❌ Using `any` type

```typescript
// BAD: Using any
export async function POST(request: NextRequest) {
  const body: any = await request.json();
}
```

### ✅ Use unknown and validate

```typescript
// GOOD: Type-safe validation
export async function POST(request: NextRequest) {
  const body: unknown = await request.json();
  const validated = RequestSchema.parse(body);
}
```
