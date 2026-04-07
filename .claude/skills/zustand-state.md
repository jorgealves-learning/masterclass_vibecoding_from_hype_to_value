---
paths:
  - "src/lib/stores/**/*.ts"
  - "src/stores/**/*.ts"
---

# Zustand State Management

> Skill for global client-side state management using Zustand (AGENTS.md §2.2.4)

---

## When to Use Zustand

Use Zustand ONLY for **global client-side state** that:

- Needs to be shared across multiple unrelated components
- Cannot be derived from server state
- Requires client-side reactivity (shopping cart, UI preferences, etc.)

**Do NOT use Zustand for:**
- Server state (use RSCs + Next.js caching)
- Local component state (use useState)
- Form state (use React Hook Form)

---

## Store Creation Pattern

```typescript
import { create } from 'zustand';

// Define state shape with explicit types
type CartItem = {
  productId: string;
  name: string;
  price: number;
  quantity: number;
};

type CartState = {
  items: CartItem[];
  isOpen: boolean;
};

type CartActions = {
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  toggleCart: () => void;
};

type CartStore = CartState & CartActions;

// Named export (AGENTS.md §3.2.4)
export const useCartStore = create<CartStore>((set) => ({
  // Initial state
  items: [],
  isOpen: false,

  // Actions
  addItem: (item) =>
    set((state) => {
      const existing = state.items.find((i) => i.productId === item.productId);
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.productId === item.productId
              ? { ...i, quantity: i.quantity + 1 }
              : i
          ),
        };
      }
      return { items: [...state.items, { ...item, quantity: 1 }] };
    }),

  removeItem: (productId) =>
    set((state) => ({
      items: state.items.filter((i) => i.productId !== productId),
    })),

  updateQuantity: (productId, quantity) =>
    set((state) => ({
      items: state.items.map((i) =>
        i.productId === productId ? { ...i, quantity } : i
      ),
    })),

  clearCart: () => set({ items: [] }),

  toggleCart: () => set((state) => ({ isOpen: !state.isOpen })),
}));
```

---

## Selector Pattern for Performance

Always use selectors to minimize re-renders:

```typescript
// ❌ BAD: Component re-renders on ANY store change
function CartBadge() {
  const store = useCartStore();
  return <span>{store.items.length}</span>;
}

// ✅ GOOD: Component only re-renders when items.length changes
function CartBadge() {
  const itemCount = useCartStore((state) => state.items.length);
  return <span>{itemCount}</span>;
}

// ✅ GOOD: Multiple selectors for specific data
function CartTotal() {
  const total = useCartStore((state) =>
    state.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  return <span>${total.toFixed(2)}</span>;
}
```

---

## Derived State with Selectors

```typescript
// Define reusable selectors
export const cartSelectors = {
  totalItems: (state: CartState) =>
    state.items.reduce((sum, item) => sum + item.quantity, 0),

  totalPrice: (state: CartState) =>
    state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),

  isEmpty: (state: CartState) => state.items.length === 0,

  itemById: (productId: string) => (state: CartState) =>
    state.items.find((item) => item.productId === productId),
};

// Usage in components
function CartSummary() {
  const totalItems = useCartStore(cartSelectors.totalItems);
  const totalPrice = useCartStore(cartSelectors.totalPrice);
  const isEmpty = useCartStore(cartSelectors.isEmpty);

  if (isEmpty) return <p>Your cart is empty</p>;

  return (
    <div>
      <p>{totalItems} items</p>
      <p>Total: ${totalPrice.toFixed(2)}</p>
    </div>
  );
}
```

---

## Persistence Middleware

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

type ThemeState = {
  theme: 'light' | 'dark' | 'system';
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
};

export const useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: 'system',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'theme-storage',
      storage: createJSONStorage(() => localStorage),
      // Only persist specific keys
      partialize: (state) => ({ theme: state.theme }),
    }
  )
);
```

---

## Immer Middleware for Complex Updates

```typescript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

type NestedState = {
  user: {
    profile: {
      name: string;
      settings: {
        notifications: boolean;
        darkMode: boolean;
      };
    };
  };
  updateNotifications: (enabled: boolean) => void;
};

export const useSettingsStore = create<NestedState>()(
  immer((set) => ({
    user: {
      profile: {
        name: '',
        settings: {
          notifications: true,
          darkMode: false,
        },
      },
    },
    // Immer allows mutable-style updates
    updateNotifications: (enabled) =>
      set((state) => {
        state.user.profile.settings.notifications = enabled;
      }),
  }))
);
```

---

## DevTools Integration

```typescript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

export const useCartStore = create<CartStore>()(
  devtools(
    (set) => ({
      items: [],
      addItem: (item) =>
        set(
          (state) => ({ items: [...state.items, { ...item, quantity: 1 }] }),
          false, // replace: false (default)
          'cart/addItem' // action name for DevTools
        ),
    }),
    { name: 'CartStore' }
  )
);
```

---

## Combining Middlewares

```typescript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

export const useStore = create<StoreType>()(
  devtools(
    persist(
      immer((set) => ({
        // state and actions
      })),
      { name: 'app-storage' }
    ),
    { name: 'AppStore' }
  )
);
```

---

## Actions Outside React

```typescript
// Access store outside of React components
const { addItem, clearCart } = useCartStore.getState();

// Subscribe to changes
const unsubscribe = useCartStore.subscribe(
  (state) => state.items,
  (items) => {
    console.log('Cart updated:', items.length, 'items');
  }
);

// Cleanup subscription
unsubscribe();
```

---

## Testing Stores

```typescript
import { act, renderHook } from '@testing-library/react';
import { useCartStore } from './cart-store';

describe('useCartStore', () => {
  beforeEach(() => {
    // Reset store between tests
    useCartStore.setState({ items: [], isOpen: false });
  });

  it('adds item to cart', () => {
    const { result } = renderHook(() => useCartStore());

    act(() => {
      result.current.addItem({
        productId: '123',
        name: 'Test Product',
        price: 29.99,
      });
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(1);
  });

  it('increments quantity for existing item', () => {
    const { result } = renderHook(() => useCartStore());

    act(() => {
      result.current.addItem({ productId: '123', name: 'Test', price: 10 });
      result.current.addItem({ productId: '123', name: 'Test', price: 10 });
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(2);
  });
});
```

---

## Anti-Patterns to Avoid

### ❌ Storing server data in Zustand

```typescript
// BAD: This should be fetched in a Server Component
export const useUsersStore = create((set) => ({
  users: [],
  fetchUsers: async () => {
    const users = await fetch('/api/users').then((r) => r.json());
    set({ users });
  },
}));
```

### ❌ Using entire store in components

```typescript
// BAD: Re-renders on ANY state change
function Component() {
  const store = useCartStore();
  return <span>{store.items.length}</span>;
}
```

### ❌ Mutating state directly

```typescript
// BAD: Direct mutation (without immer middleware)
addItem: (item) => set((state) => {
  state.items.push(item); // WRONG!
  return state;
}),
```

### ✅ Correct patterns

```typescript
// GOOD: Use selectors
const itemCount = useCartStore((state) => state.items.length);

// GOOD: Return new state objects
addItem: (item) => set((state) => ({
  items: [...state.items, item],
})),
```

---

## File Organization

```
src/
└── lib/
    └── stores/
        ├── cart.ts        # Cart store
        ├── theme.ts       # Theme preferences store
        ├── ui.ts          # UI state (modals, sidebars)
        └── index.ts       # Re-export all stores
```

```typescript
// src/lib/stores/index.ts
export { useCartStore, cartSelectors } from './cart';
export { useThemeStore } from './theme';
export { useUIStore } from './ui';
```
