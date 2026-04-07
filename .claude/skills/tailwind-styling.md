---
paths:
  - "src/**/*.tsx"
  - "src/components/**/*"
  - "tailwind.config.*"
---

# Tailwind CSS & Styling Skills

> Skill for implementing styles following AGENTS.md mandated stack (Tailwind CSS, Radix UI, shadcn/ui)

---

## Core Principles

1. **Utility-First**: Use Tailwind utilities directly in components
2. **Component Abstraction**: Extract repeated patterns into components, not CSS classes
3. **Design Tokens**: Use theme values, not arbitrary values
4. **Responsive by Default**: Mobile-first responsive design

---

## Tailwind Configuration

### Standard Configuration

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: 'class',
  content: [
    './src/**/*.{ts,tsx}',
    './src/components/**/*.{ts,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      fontFamily: {
        sans: ['var(--font-sans)', 'system-ui', 'sans-serif'],
        mono: ['var(--font-mono)', 'monospace'],
      },
      keyframes: {
        'accordion-down': {
          from: { height: '0' },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: '0' },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};

export default config;
```

### CSS Variables Setup

```css
/* src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
  }
}
```

---

## Component Styling Patterns

### Basic Component with Variants

```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants>;

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  );
}
```

### The `cn` Utility Function

```typescript
// src/lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs));
}
```

---

## Responsive Design Patterns

### Mobile-First Approach

```tsx
// ✅ CORRECT: Mobile-first, progressively enhance
<div className="
  flex flex-col          {/* Mobile: stack vertically */}
  md:flex-row            {/* Tablet+: horizontal layout */}
  gap-4                  {/* Consistent spacing */}
  p-4 md:p-6 lg:p-8      {/* Progressive padding */}
">
  <aside className="
    w-full               {/* Mobile: full width */}
    md:w-64              {/* Tablet+: fixed sidebar */}
    lg:w-80              {/* Desktop: wider sidebar */}
  ">
    Sidebar
  </aside>
  <main className="flex-1">
    Content
  </main>
</div>
```

### Responsive Typography

```tsx
<h1 className="
  text-2xl               {/* Mobile */}
  sm:text-3xl            {/* Small screens */}
  md:text-4xl            {/* Medium screens */}
  lg:text-5xl            {/* Large screens */}
  font-bold tracking-tight
">
  Heading
</h1>
```

### Responsive Grid

```tsx
<div className="
  grid
  grid-cols-1            {/* Mobile: 1 column */}
  sm:grid-cols-2         {/* Small: 2 columns */}
  lg:grid-cols-3         {/* Large: 3 columns */}
  xl:grid-cols-4         {/* Extra large: 4 columns */}
  gap-4 md:gap-6
">
  {items.map(item => (
    <Card key={item.id} item={item} />
  ))}
</div>
```

---

## Dark Mode Implementation

### Theme Provider Setup

```tsx
// src/components/theme-provider.tsx
'use client';

import { ThemeProvider as NextThemesProvider } from 'next-themes';
import { type ThemeProviderProps } from 'next-themes/dist/types';

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>;
}

// Usage in layout
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### Theme Toggle Component

```tsx
'use client';

import { Moon, Sun } from 'lucide-react';
import { useTheme } from 'next-themes';
import { Button } from '@/components/ui/button';

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">Toggle theme</span>
    </Button>
  );
}
```

### Dark Mode Styles

```tsx
// Use dark: prefix for dark mode variants
<div className="
  bg-white dark:bg-gray-900
  text-gray-900 dark:text-gray-100
  border-gray-200 dark:border-gray-700
">
  Content that adapts to theme
</div>

// Or use CSS variables (preferred for consistency)
<div className="bg-background text-foreground border-border">
  Uses theme-aware CSS variables
</div>
```

---

## shadcn/ui Patterns

### Installing Components

```bash
# Initialize shadcn/ui
npx shadcn@latest init

# Add specific components
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add dialog
```

### Using shadcn/ui Components

```tsx
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { Button } from '@/components/ui/button';

export function ProductCard({ product }: { product: Product }) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{product.name}</CardTitle>
        <CardDescription>{product.category}</CardDescription>
      </CardHeader>
      <CardContent>
        <p className="text-2xl font-bold">${product.price}</p>
        <p className="text-muted-foreground">{product.description}</p>
      </CardContent>
      <CardFooter>
        <Button className="w-full">Add to Cart</Button>
      </CardFooter>
    </Card>
  );
}
```

### Customizing shadcn/ui Components

```tsx
// Extend the base component with custom variants
import { Button, buttonVariants } from '@/components/ui/button';
import { cn } from '@/lib/utils';

// Option 1: Wrapper component
export function PrimaryButton({ className, ...props }: ButtonProps) {
  return (
    <Button
      className={cn('bg-brand-500 hover:bg-brand-600', className)}
      {...props}
    />
  );
}

// Option 2: Use as className utility
<a
  href="/login"
  className={cn(
    buttonVariants({ variant: 'outline', size: 'lg' }),
    'no-underline'
  )}
>
  Login
</a>
```

---

## Layout Patterns

### Container Pattern

```tsx
// Consistent max-width container
<div className="container mx-auto px-4 sm:px-6 lg:px-8">
  {children}
</div>

// Or define in tailwind.config.ts
// theme: { container: { center: true, padding: '2rem' } }
```

### Stack Pattern

```tsx
// Vertical stack
<div className="flex flex-col gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

// Horizontal stack with wrapping
<div className="flex flex-wrap gap-2">
  {tags.map(tag => (
    <span key={tag} className="rounded-full bg-muted px-3 py-1 text-sm">
      {tag}
    </span>
  ))}
</div>
```

### Sticky Header Pattern

```tsx
<header className="
  sticky top-0 z-50
  border-b border-border
  bg-background/95 backdrop-blur
  supports-[backdrop-filter]:bg-background/60
">
  <nav className="container flex h-16 items-center">
    {/* Navigation content */}
  </nav>
</header>
```

---

## Animation Patterns

### Tailwind Transitions

```tsx
// Hover transitions
<button className="
  transition-colors duration-200
  hover:bg-primary hover:text-primary-foreground
">
  Hover me
</button>

// Transform transitions
<div className="
  transition-transform duration-300
  hover:scale-105 hover:-translate-y-1
">
  Card with lift effect
</div>
```

### Using tailwindcss-animate

```tsx
// Fade in animation
<div className="animate-in fade-in duration-500">
  Fades in on mount
</div>

// Slide in from direction
<div className="animate-in slide-in-from-bottom-4 duration-300">
  Slides up on mount
</div>

// Combining animations
<div className="animate-in fade-in slide-in-from-right-4 duration-500">
  Fades and slides
</div>
```

---

## Anti-Patterns to Avoid

### ❌ Using @apply Excessively

```css
/* BAD: Creating CSS classes defeats utility-first purpose */
.btn-primary {
  @apply bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600;
}
```

```tsx
// ✅ GOOD: Use component abstraction instead
export function PrimaryButton({ children, ...props }: ButtonProps) {
  return (
    <button
      className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
      {...props}
    >
      {children}
    </button>
  );
}
```

### ❌ Arbitrary Values Instead of Theme

```tsx
// BAD: Arbitrary values create inconsistency
<div className="text-[13px] p-[17px] bg-[#3B82F6]">

// GOOD: Use theme values
<div className="text-sm p-4 bg-primary">
```

### ❌ Inline Styles

```tsx
// BAD: Mixing inline styles with Tailwind
<div style={{ marginTop: '20px' }} className="p-4">

// GOOD: Use Tailwind utilities
<div className="mt-5 p-4">
```

### ❌ Not Using cn() for Conditional Classes

```tsx
// BAD: String concatenation is error-prone
<div className={'p-4 ' + (isActive ? 'bg-primary' : 'bg-secondary')}>

// GOOD: Use cn() utility
<div className={cn('p-4', isActive ? 'bg-primary' : 'bg-secondary')}>
```

---

## Performance Optimization

### Purging Unused CSS

Tailwind automatically purges unused styles in production. Ensure `content` paths are correct:

```typescript
// tailwind.config.ts
content: [
  './src/**/*.{ts,tsx}',
  './src/components/**/*.{ts,tsx}',
  // Include any other paths where you use Tailwind classes
],
```

### Avoiding Dynamic Class Names

```tsx
// ❌ BAD: Dynamic class names won't be detected by purge
const color = 'red';
<div className={`text-${color}-500`}> // Won't work!

// ✅ GOOD: Use complete class names
const colorClasses = {
  red: 'text-red-500',
  blue: 'text-blue-500',
  green: 'text-green-500',
} as const;

<div className={colorClasses[color]}>
```

---

## Accessibility Considerations

```tsx
// Focus states for keyboard navigation
<button className="
  focus:outline-none
  focus-visible:ring-2
  focus-visible:ring-primary
  focus-visible:ring-offset-2
">
  Accessible button
</button>

// Screen reader only text
<span className="sr-only">Loading, please wait</span>

// Reduced motion support
<div className="
  animate-bounce
  motion-reduce:animate-none
">
  Bouncing element (respects prefers-reduced-motion)
</div>
```
