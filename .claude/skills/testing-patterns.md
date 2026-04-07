---
paths:
  - "src/**/*.test.ts"
  - "src/**/*.test.tsx"
  - "src/**/*.spec.ts"
  - "src/**/*.spec.tsx"
  - "tests/**/*"
  - "e2e/**/*"
---

# Testing Patterns

> Skill for implementing tests following AGENTS.md Section 4.4 mandates

---

## Unit Testing with Jest & React Testing Library

### Component Testing Pattern

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { ContactForm } from './ContactForm';

describe('ContactForm', () => {
  it('renders all form fields', () => {
    render(<ContactForm />);
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/message/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  it('validates required fields', async () => {
    const user = userEvent.setup();
    render(<ContactForm />);
    
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
  });

  it('submits valid form data', async () => {
    const onSubmit = vi.fn();
    const user = userEvent.setup();
    render(<ContactForm onSubmit={onSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/message/i), 'Hello world');
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        message: 'Hello world',
      });
    });
  });
});
```

### Hook Testing Pattern

```tsx
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments counter', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('decrements counter', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
});
```

### Business Logic Testing

```typescript
import { describe, it, expect } from 'vitest';
import { calculateDiscount, formatCurrency, validateEmail } from './utils';

describe('calculateDiscount', () => {
  it('calculates percentage discount correctly', () => {
    expect(calculateDiscount(100, 10)).toBe(90);
    expect(calculateDiscount(100, 25)).toBe(75);
    expect(calculateDiscount(50, 50)).toBe(25);
  });

  it('handles zero discount', () => {
    expect(calculateDiscount(100, 0)).toBe(100);
  });

  it('handles 100% discount', () => {
    expect(calculateDiscount(100, 100)).toBe(0);
  });

  it('throws for invalid percentage', () => {
    expect(() => calculateDiscount(100, -10)).toThrow('Invalid percentage');
    expect(() => calculateDiscount(100, 110)).toThrow('Invalid percentage');
  });
});

describe('formatCurrency', () => {
  it('formats USD correctly', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56');
  });

  it('formats EUR correctly', () => {
    expect(formatCurrency(1234.56, 'EUR')).toBe('€1,234.56');
  });

  it('handles zero', () => {
    expect(formatCurrency(0, 'USD')).toBe('$0.00');
  });
});

describe('validateEmail', () => {
  it('accepts valid emails', () => {
    expect(validateEmail('user@example.com')).toBe(true);
    expect(validateEmail('user.name@example.co.uk')).toBe(true);
  });

  it('rejects invalid emails', () => {
    expect(validateEmail('notanemail')).toBe(false);
    expect(validateEmail('@example.com')).toBe(false);
    expect(validateEmail('user@')).toBe(false);
  });
});
```

### Zod Schema Testing

```typescript
import { describe, it, expect } from 'vitest';
import { userSchema, createUserSchema } from './schemas';

describe('userSchema', () => {
  it('accepts valid user data', () => {
    const validData = {
      id: '123e4567-e89b-12d3-a456-426614174000',
      email: 'user@example.com',
      name: 'John Doe',
    };
    
    const result = userSchema.safeParse(validData);
    expect(result.success).toBe(true);
  });

  it('rejects invalid email', () => {
    const invalidData = {
      id: '123e4567-e89b-12d3-a456-426614174000',
      email: 'not-an-email',
      name: 'John Doe',
    };
    
    const result = userSchema.safeParse(invalidData);
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.flatten().fieldErrors.email).toBeDefined();
    }
  });

  it('rejects missing required fields', () => {
    const result = userSchema.safeParse({});
    expect(result.success).toBe(false);
  });
});
```

### API Route Handler Testing

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { NextRequest } from 'next/server';
import { POST, GET } from './route';

// Mock dependencies
vi.mock('@/lib/services/user', () => ({
  userService: {
    create: vi.fn(),
    findById: vi.fn(),
  },
}));

import { userService } from '@/lib/services/user';

describe('POST /api/users', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('creates user with valid data', async () => {
    const mockUser = { id: '1', email: 'test@example.com', name: 'Test' };
    vi.mocked(userService.create).mockResolvedValue(mockUser);

    const request = new NextRequest('http://localhost/api/users', {
      method: 'POST',
      body: JSON.stringify({ email: 'test@example.com', name: 'Test' }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(201);
    expect(data).toEqual(mockUser);
    expect(userService.create).toHaveBeenCalledWith({
      email: 'test@example.com',
      name: 'Test',
    });
  });

  it('returns 400 for invalid data', async () => {
    const request = new NextRequest('http://localhost/api/users', {
      method: 'POST',
      body: JSON.stringify({ email: 'invalid-email' }),
    });

    const response = await POST(request);
    
    expect(response.status).toBe(400);
    expect(userService.create).not.toHaveBeenCalled();
  });

  it('returns 400 for invalid JSON', async () => {
    const request = new NextRequest('http://localhost/api/users', {
      method: 'POST',
      body: 'not-json',
    });

    const response = await POST(request);
    
    expect(response.status).toBe(400);
  });
});
```

### Mocking Patterns

```typescript
import { vi } from 'vitest';

// Mock external module
vi.mock('@/lib/database', () => ({
  db: {
    query: vi.fn(),
    transaction: vi.fn(),
  },
}));

// Mock fetch
const mockFetch = vi.fn();
global.fetch = mockFetch;

beforeEach(() => {
  mockFetch.mockReset();
});

it('fetches data correctly', async () => {
  mockFetch.mockResolvedValueOnce({
    ok: true,
    json: () => Promise.resolve({ data: 'test' }),
  });

  const result = await fetchData();
  
  expect(mockFetch).toHaveBeenCalledWith('/api/data');
  expect(result).toEqual({ data: 'test' });
});

// Mock timers
it('debounces input', async () => {
  vi.useFakeTimers();
  const callback = vi.fn();
  const debouncedFn = debounce(callback, 300);

  debouncedFn('first');
  debouncedFn('second');
  debouncedFn('third');

  expect(callback).not.toHaveBeenCalled();
  
  vi.advanceTimersByTime(300);
  
  expect(callback).toHaveBeenCalledTimes(1);
  expect(callback).toHaveBeenCalledWith('third');
  
  vi.useRealTimers();
});
```

---

## End-to-End Testing with Playwright

### Basic E2E Test

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Authentication', () => {
  test('allows user to log in', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[data-testid="email-input"]', 'user@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    await page.click('[data-testid="submit-button"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome-message"]')).toContainText('Welcome');
  });

  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[data-testid="email-input"]', 'user@example.com');
    await page.fill('[data-testid="password-input"]', 'wrongpassword');
    await page.click('[data-testid="submit-button"]');
    
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
    await expect(page).toHaveURL('/login');
  });
});
```

### Authentication Setup

```typescript
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[data-testid="email-input"]', process.env.TEST_USER_EMAIL!);
  await page.fill('[data-testid="password-input"]', process.env.TEST_USER_PASSWORD!);
  await page.click('[data-testid="submit-button"]');
  
  await expect(page).toHaveURL('/dashboard');
  
  await page.context().storageState({ path: authFile });
});
```

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    { name: 'setup', testMatch: /.*\.setup\.ts/ },
    {
      name: 'chromium',
      use: {
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
  ],
});
```

### Page Object Model

```typescript
// tests/pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.submitButton = page.locator('[data-testid="submit-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string): Promise<void> {
    await expect(this.errorMessage).toContainText(message);
  }
}

// Usage in test
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('login flow', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  await expect(page).toHaveURL('/dashboard');
});
```

### Visual Regression Testing

```typescript
import { test, expect } from '@playwright/test';

test('homepage visual snapshot', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png');
});

test('component visual snapshot', async ({ page }) => {
  await page.goto('/components/button');
  const button = page.locator('[data-testid="primary-button"]');
  await expect(button).toHaveScreenshot('primary-button.png');
});
```

### Accessibility Testing

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('homepage has no accessibility violations', async ({ page }) => {
  await page.goto('/');
  
  const results = await new AxeBuilder({ page }).analyze();
  
  expect(results.violations).toEqual([]);
});

test('form has proper ARIA labels', async ({ page }) => {
  await page.goto('/contact');
  
  const results = await new AxeBuilder({ page })
    .include('[data-testid="contact-form"]')
    .analyze();
  
  expect(results.violations).toEqual([]);
});
```

---

## Test Data Management

### Factory Pattern

```typescript
// tests/factories/user.ts
import { faker } from '@faker-js/faker';
import type { User, CreateUserInput } from '@/types';

export function createUserInput(overrides?: Partial<CreateUserInput>): CreateUserInput {
  return {
    email: faker.internet.email(),
    name: faker.person.fullName(),
    ...overrides,
  };
}

export function createUser(overrides?: Partial<User>): User {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    name: faker.person.fullName(),
    createdAt: faker.date.past(),
    updatedAt: faker.date.recent(),
    ...overrides,
  };
}

// Usage
const user = createUser({ email: 'specific@example.com' });
const input = createUserInput();
```

### Test Fixtures

```typescript
// tests/fixtures/users.ts
export const testUsers = {
  admin: {
    id: '1',
    email: 'admin@example.com',
    name: 'Admin User',
    role: 'admin',
  },
  regular: {
    id: '2',
    email: 'user@example.com',
    name: 'Regular User',
    role: 'user',
  },
} as const;
```

---

## Anti-Patterns to Avoid

### ❌ Testing implementation details

```typescript
// BAD: Testing internal state
it('sets loading to true', () => {
  const { result } = renderHook(() => useData());
  act(() => result.current.fetch());
  expect(result.current.state.loading).toBe(true); // Implementation detail
});

// GOOD: Testing behavior
it('shows loading indicator while fetching', async () => {
  render(<DataList />);
  fireEvent.click(screen.getByRole('button', { name: /refresh/i }));
  expect(screen.getByRole('progressbar')).toBeInTheDocument();
});
```

### ❌ Hardcoded test data with PII

```typescript
// BAD: Using real-looking PII
const user = { email: 'john.smith@gmail.com', phone: '555-123-4567' };

// GOOD: Using obviously fake data
const user = { email: 'test@example.com', phone: '000-000-0000' };
// Or use faker
const user = { email: faker.internet.email(), phone: faker.phone.number() };
```

### ❌ Flaky tests with timeouts

```typescript
// BAD: Arbitrary timeouts
await new Promise(resolve => setTimeout(resolve, 1000));
expect(element).toBeVisible();

// GOOD: Wait for specific conditions
await waitFor(() => expect(element).toBeVisible());
```

---

## Coverage Requirements

- All business logic: 90%+ coverage
- Complex components: 80%+ coverage
- Critical user flows: 100% E2E coverage
- API routes: 100% coverage for happy path + error cases
