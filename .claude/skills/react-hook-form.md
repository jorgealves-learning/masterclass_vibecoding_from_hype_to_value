---
paths:
  - "src/**/*.tsx"
  - "src/components/**/*"
---

# React Hook Form Skills

> Skill for form handling following AGENTS.md mandated stack (React Hook Form + Zod)

---

## Core Integration Pattern

React Hook Form is the mandated library for form state management. Always integrate with Zod for validation.

### Basic Form Setup

```tsx
// 'use client' justification: Form requires controlled inputs and submission handling
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 1. Define schema with Zod
const formSchema = z.object({
  email: z.string().email('Please enter a valid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

// 2. Infer TypeScript type from schema
type FormData = z.infer<typeof formSchema>;

// 3. Create form component
export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(formSchema),
  });

  const onSubmit = async (data: FormData) => {
    // data is fully typed and validated
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} type="email" placeholder="Email" />
        {errors.email && <span className="text-red-500">{errors.email.message}</span>}
      </div>

      <div>
        <input {...register('password')} type="password" placeholder="Password" />
        {errors.password && <span className="text-red-500">{errors.password.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
}
```

---

## Form State Management

### Accessing Form State

```tsx
const {
  register,           // Register inputs
  handleSubmit,       // Handle form submission
  formState: {
    errors,           // Validation errors
    isSubmitting,     // Form is being submitted
    isValid,          // Form passes validation
    isDirty,          // Any field has been modified
    dirtyFields,      // Object of modified fields
    touchedFields,    // Object of touched fields
  },
  watch,              // Watch field values
  setValue,           // Set field value programmatically
  getValues,          // Get current values
  reset,              // Reset form to defaults
  setError,           // Set error manually
  clearErrors,        // Clear errors
  trigger,            // Trigger validation
} = useForm<FormData>({
  resolver: zodResolver(formSchema),
  defaultValues: {
    email: '',
    password: '',
  },
});
```

### Watching Field Values

```tsx
export function DynamicForm() {
  const { register, watch } = useForm<FormData>();

  // Watch single field
  const email = watch('email');

  // Watch multiple fields
  const [firstName, lastName] = watch(['firstName', 'lastName']);

  // Watch all fields
  const allValues = watch();

  // Watch with callback (for side effects)
  useEffect(() => {
    const subscription = watch((value, { name, type }) => {
      console.log('Field changed:', name, value);
    });
    return () => subscription.unsubscribe();
  }, [watch]);

  return (
    <form>
      <input {...register('email')} />
      <p>Preview: {email}</p>
    </form>
  );
}
```

---

## Complex Form Patterns

### Nested Objects

```tsx
const addressSchema = z.object({
  user: z.object({
    name: z.string().min(2),
    email: z.string().email(),
  }),
  address: z.object({
    street: z.string().min(5),
    city: z.string().min(2),
    zipCode: z.string().regex(/^\d{5}$/, 'Invalid ZIP code'),
  }),
});

type AddressFormData = z.infer<typeof addressSchema>;

export function AddressForm() {
  const { register, formState: { errors } } = useForm<AddressFormData>({
    resolver: zodResolver(addressSchema),
  });

  return (
    <form>
      {/* Nested field registration */}
      <input {...register('user.name')} placeholder="Name" />
      {errors.user?.name && <span>{errors.user.name.message}</span>}

      <input {...register('user.email')} placeholder="Email" />
      {errors.user?.email && <span>{errors.user.email.message}</span>}

      <input {...register('address.street')} placeholder="Street" />
      {errors.address?.street && <span>{errors.address.street.message}</span>}

      <input {...register('address.city')} placeholder="City" />
      <input {...register('address.zipCode')} placeholder="ZIP" />
    </form>
  );
}
```

### Dynamic Field Arrays

```tsx
import { useFieldArray } from 'react-hook-form';

const orderSchema = z.object({
  items: z.array(
    z.object({
      productId: z.string().min(1),
      quantity: z.number().int().positive(),
    })
  ).min(1, 'At least one item required'),
});

type OrderFormData = z.infer<typeof orderSchema>;

export function OrderForm() {
  const { register, control, handleSubmit, formState: { errors } } = useForm<OrderFormData>({
    resolver: zodResolver(orderSchema),
    defaultValues: {
      items: [{ productId: '', quantity: 1 }],
    },
  });

  const { fields, append, remove, move } = useFieldArray({
    control,
    name: 'items',
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input
            {...register(`items.${index}.productId`)}
            placeholder="Product ID"
          />
          <input
            {...register(`items.${index}.quantity`, { valueAsNumber: true })}
            type="number"
            min="1"
          />
          <button type="button" onClick={() => remove(index)}>
            Remove
          </button>
        </div>
      ))}

      {errors.items && <span>{errors.items.message}</span>}

      <button type="button" onClick={() => append({ productId: '', quantity: 1 })}>
        Add Item
      </button>
      <button type="submit">Submit Order</button>
    </form>
  );
}
```

---

## Async Validation

```tsx
const usernameSchema = z.object({
  username: z.string()
    .min(3, 'Username must be at least 3 characters')
    .refine(async (username) => {
      // Check if username is available
      const response = await fetch(`/api/check-username?username=${username}`);
      const { available } = await response.json();
      return available;
    }, 'Username is already taken'),
});

export function UsernameForm() {
  const { register, handleSubmit, formState: { errors, isValidating } } = useForm({
    resolver: zodResolver(usernameSchema),
    mode: 'onBlur', // Validate on blur for async
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('username')} />
      {isValidating && <span>Checking...</span>}
      {errors.username && <span>{errors.username.message}</span>}
    </form>
  );
}
```

---

## Server Actions Integration

```tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useTransition } from 'react';
import { createUser } from '@/lib/actions/user';
import { CreateUserSchema, type CreateUserInput } from '@/lib/schemas/user';

export function CreateUserForm() {
  const [isPending, startTransition] = useTransition();

  const {
    register,
    handleSubmit,
    formState: { errors },
    setError,
    reset,
  } = useForm<CreateUserInput>({
    resolver: zodResolver(CreateUserSchema),
  });

  const onSubmit = (data: CreateUserInput) => {
    startTransition(async () => {
      const result = await createUser(data);

      if (result.success) {
        reset();
        // Show success message or redirect
      } else {
        // Map server errors to form fields
        if (result.fieldErrors) {
          Object.entries(result.fieldErrors).forEach(([field, message]) => {
            setError(field as keyof CreateUserInput, { message });
          });
        } else {
          setError('root', { message: result.error });
        }
      }
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}

      {errors.root && <span className="text-red-500">{errors.root.message}</span>}

      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

---

## Controlled Components Integration

### With Custom Input Components

```tsx
import { Controller, useForm } from 'react-hook-form';

export function FormWithCustomInputs() {
  const { control, handleSubmit } = useForm<FormData>();

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="category"
        control={control}
        render={({ field, fieldState: { error } }) => (
          <div>
            <Select
              value={field.value}
              onValueChange={field.onChange}
              onBlur={field.onBlur}
            >
              <SelectTrigger>
                <SelectValue placeholder="Select category" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="electronics">Electronics</SelectItem>
                <SelectItem value="clothing">Clothing</SelectItem>
              </SelectContent>
            </Select>
            {error && <span>{error.message}</span>}
          </div>
        )}
      />
    </form>
  );
}
```

### With Radix UI Components

```tsx
import { Controller } from 'react-hook-form';
import * as Checkbox from '@radix-ui/react-checkbox';

export function TermsForm() {
  const { control, handleSubmit } = useForm({
    defaultValues: { acceptTerms: false },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="acceptTerms"
        control={control}
        rules={{ required: 'You must accept the terms' }}
        render={({ field }) => (
          <div className="flex items-center gap-2">
            <Checkbox.Root
              checked={field.value}
              onCheckedChange={field.onChange}
              className="h-5 w-5 rounded border"
            >
              <Checkbox.Indicator>✓</Checkbox.Indicator>
            </Checkbox.Root>
            <label>I accept the terms and conditions</label>
          </div>
        )}
      />
    </form>
  );
}
```

---

## Error Display Patterns

### Inline Errors

```tsx
function FormField({ name, label, register, errors }: FieldProps) {
  return (
    <div className="space-y-1">
      <label htmlFor={name} className="text-sm font-medium">
        {label}
      </label>
      <input
        id={name}
        {...register(name)}
        className={cn(
          'w-full rounded border px-3 py-2',
          errors[name] ? 'border-red-500' : 'border-gray-300'
        )}
        aria-invalid={errors[name] ? 'true' : 'false'}
        aria-describedby={errors[name] ? `${name}-error` : undefined}
      />
      {errors[name] && (
        <p id={`${name}-error`} className="text-sm text-red-500" role="alert">
          {errors[name].message}
        </p>
      )}
    </div>
  );
}
```

### Summary Errors

```tsx
function ErrorSummary({ errors }: { errors: FieldErrors }) {
  const errorMessages = Object.entries(errors)
    .filter(([, error]) => error?.message)
    .map(([field, error]) => ({ field, message: error!.message }));

  if (errorMessages.length === 0) return null;

  return (
    <div className="rounded border border-red-500 bg-red-50 p-4" role="alert">
      <h3 className="font-medium text-red-800">Please fix the following errors:</h3>
      <ul className="mt-2 list-inside list-disc text-red-700">
        {errorMessages.map(({ field, message }) => (
          <li key={field}>{message}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Validation Modes

```tsx
useForm({
  resolver: zodResolver(schema),
  mode: 'onChange',      // Validate on every change (expensive)
  mode: 'onBlur',        // Validate on blur (recommended for async)
  mode: 'onSubmit',      // Validate only on submit (default)
  mode: 'onTouched',     // Validate on blur, then on change after error
  mode: 'all',           // Validate on blur and change
  
  reValidateMode: 'onChange', // When to re-validate after error
});
```

---

## Anti-Patterns to Avoid

### ❌ Not Using Zod Resolver

```tsx
// BAD: Manual validation rules
<input {...register('email', {
  required: 'Email required',
  pattern: { value: /^\S+@\S+$/, message: 'Invalid email' },
})} />

// GOOD: Zod schema
const schema = z.object({
  email: z.string().email('Invalid email'),
});

useForm({ resolver: zodResolver(schema) });
```

### ❌ Uncontrolled Selects Without defaultValue

```tsx
// BAD: Select won't have initial value
<select {...register('country')}>
  <option value="">Select country</option>
  <option value="us">United States</option>
</select>

// GOOD: Provide defaultValues
useForm({
  defaultValues: { country: '' },
});
```

### ❌ Not Handling Server Errors

```tsx
// BAD: Ignoring server validation errors
const onSubmit = async (data) => {
  await createUser(data); // What if server returns errors?
};

// GOOD: Map server errors to form
const onSubmit = async (data) => {
  const result = await createUser(data);
  if (!result.success) {
    setError('email', { message: result.error });
  }
};
```

---

## Performance Optimization

```tsx
// Use shouldUnregister: false to preserve values
useForm({
  shouldUnregister: false, // Keep values when fields unmount
});

// Isolate re-renders with Controller
<Controller
  name="expensiveField"
  control={control}
  render={({ field }) => <ExpensiveComponent {...field} />}
/>

// Use watch sparingly - prefer getValues for one-time reads
const handleClick = () => {
  const values = getValues(); // Doesn't cause re-render
};
```
