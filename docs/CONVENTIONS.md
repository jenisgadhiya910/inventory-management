# Coding Conventions & Patterns

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `ProductTable.tsx`, `StepIndicator.tsx` |
| Hooks | camelCase with `use` prefix | `useProducts.ts`, `useDebounce.ts` |
| Contexts | PascalCase with `Context`/`Provider` suffix | `ProductContext.tsx`, `ProductProvider` |
| Utilities | camelCase | `formatCurrency.ts`, `generateId.ts` |
| Types/Interfaces | PascalCase, no `I` prefix | `Product`, `StockAlert`, `ProductFilters` |
| Constants | UPPER_SNAKE_CASE | `STORAGE_KEYS`, `DEFAULT_CATEGORIES` |
| Event handlers | `handle` prefix in components, `on` prefix in props | `handleSubmit`, `onClick` |
| Boolean variables | `is`/`has`/`should` prefix | `isLoading`, `hasError`, `shouldRefetch` |
| Files | Match the default export name | `ProductTable.tsx` exports `ProductTable` |

---

## Component Patterns

### Functional Components Only

All components are functional components with hooks. No class components. **Preferred style: named `export function` declarations** (not arrow functions) for consistency and better debugging stack traces.

```typescript
// Preferred
export function ProductRow({ product, onEdit }: ProductRowProps) {
  return <tr>...</tr>;
}

// Avoid arrow function exports — use only for tiny inline components within a file
const StatusDot = ({ color }: { color: string }) => (
  <div className="h-2 w-2 rounded-full" style={{ backgroundColor: color }} />
);
```

### React 19 Notes

- **Do NOT** `import React from 'react'` — JSX transform is automatic (configured in `tsconfig.json`).
- Import only what you use: `import { useState, useMemo } from 'react'`.
- React 19's `use()` hook is available but we use `useContext()` for consistency across the codebase.

### Props Interface

Define props inline for simple components, as a named interface for complex ones:

```typescript
// Simple — inline
export function StatusBadge({ status }: { status: StockStatus }) { ... }

// Complex — named interface, co-located above the component
interface ProductTableProps {
  products: Product[];
  filters: ProductFilters;
  onProductSelect: (id: string) => void;
  onBulkAction: (action: BulkAction, ids: string[]) => void;
}

export function ProductTable({ products, filters, onProductSelect, onBulkAction }: ProductTableProps) { ... }
```

### Component Composition

Prefer composition over configuration:

```typescript
// Prefer this
<Dialog>
  <DialogTrigger asChild>
    <Button>Edit</Button>
  </DialogTrigger>
  <DialogContent>
    <ProductForm product={product} onSave={handleSave} />
  </DialogContent>
</Dialog>

// Over this
<ProductEditDialog isOpen={isOpen} product={product} onSave={handleSave} onClose={handleClose} />
```

---

## Hook Patterns

### Custom Hook Structure

```typescript
export function useProducts() {
  const { state, dispatch } = useContext(ProductContext);

  // Derived/computed values
  const lowStockProducts = useMemo(
    () => state.products.filter(p => p.quantity <= p.minStockLevel && p.quantity > 0),
    [state.products]
  );

  // Actions
  const addProduct = useCallback((data: CreateProductInput) => {
    const product = buildProduct(data);
    dispatch({ type: 'ADD_PRODUCT', payload: product });
    return product;
  }, [dispatch]);

  // Return object — consistent shape
  return {
    products: state.products,
    lowStockProducts,
    loading: state.loading,
    addProduct,
    updateProduct,
    deleteProduct,
  };
}
```

### Hook Rules

- Hooks must be in `hooks/` directory (feature-level or app-level).
- One hook per file. File name matches hook name.
- Hooks return objects, not arrays (except for simple two-value hooks like `useDebounce`).
- Hooks never call `localStorage` directly — they go through context/providers.

---

## TypeScript Patterns

### Strict Mode

`tsconfig.json` has `"strict": true`. This means:
- No implicit `any`
- Strict null checks enabled
- All function parameters must be typed

### Type Exports

All shared types live in `src/types/index.ts`. Feature-specific types can live in `features/<feature>/types.ts`.

```typescript
// src/types/index.ts
export interface Product { ... }
export interface Category { ... }
export interface Supplier { ... }
export interface StockAlert { ... }
export interface StockTransaction { ... }

// Derived types
export type StockStatus = 'in_stock' | 'low_stock' | 'out_of_stock';
export type AlertSeverity = 'info' | 'warning' | 'critical';
export type TransactionType = 'restock' | 'sale' | 'adjustment' | 'return';

// Form input types (omit auto-generated fields)
export type CreateProductInput = Omit<Product, 'id' | 'createdAt' | 'updatedAt'>;
export type UpdateProductInput = Partial<CreateProductInput> & { id: string };
```

### Avoid `any`

Use `unknown` for truly unknown types, then narrow with type guards:

```typescript
function parseStoredData<T>(raw: string | null, fallback: T): T {
  if (!raw) return fallback;
  try {
    return JSON.parse(raw) as T;
  } catch {
    return fallback;
  }
}
```

---

## Styling Patterns

### Tailwind CSS v4

Use Tailwind utility classes directly on elements. No custom CSS unless absolutely necessary.

```typescript
// Correct
<div className="flex items-center gap-2 rounded-lg border p-4">

// Avoid — don't create CSS modules or styled-components
```

### Conditional Classes

Use `cn()` utility (from shadcn/ui's `lib/utils.ts`) for conditional classes:

```typescript
import { cn } from '@/lib/utils';

<div className={cn(
  'rounded-lg border p-4',
  isActive && 'border-primary bg-primary/5',
  isDisabled && 'opacity-50 pointer-events-none'
)}>
```

### Responsive Design

Use Tailwind breakpoints. Mobile-first approach:

```typescript
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4">
```

| Breakpoint | Width | Target |
|-----------|-------|--------|
| (default) | < 768px | Mobile |
| `md:` | >= 768px | Tablet |
| `lg:` | >= 1024px | Desktop |

---

## File Organization Rules

1. **One component per file** — except tightly coupled sub-components (e.g., `TableRow` inside `Table.tsx`).
2. **Index files** — Use `index.ts` barrel exports only at the feature level, not in every folder.
3. **Co-location** — Tests sit next to the file they test: `ProductTable.tsx` + `ProductTable.test.tsx`.
4. **No circular imports** — Features depend on shared types and hooks, never on each other's components.

---

## Import Order

```typescript
// 1. React and framework imports
import { useState, useMemo } from 'react';
import { useNavigate, useParams } from 'react-router-dom';

// 2. Third-party libraries
import { format } from 'date-fns';
import { BarChart, Bar, XAxis, YAxis } from 'recharts';

// 3. UI components (shadcn/ui)
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

// 4. Feature components and hooks
import { useProducts } from '../hooks/useProducts';
import { ProductRow } from './ProductRow';

// 5. Types
import type { Product, ProductFilters } from '@/types';

// 6. Utils and constants
import { formatCurrency } from '@/lib/format';
import { STORAGE_KEYS } from '@/lib/storage';
```

---

## Error Handling Patterns

### User-Facing Errors

Use `sonner` (toast) for transient feedback:

```typescript
import { toast } from 'sonner';

// Success
toast.success('Product added successfully');

// Error
toast.error('Failed to save product. Please try again.');

// With undo
toast('Product deleted', {
  action: { label: 'Undo', onClick: () => restoreProduct(id) },
  duration: 5000,
});
```

### Data Validation Errors

Use **shadcn/ui Form components** integrated with React Hook Form + Zod:

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';

const form = useForm<ProductFormValues>({
  resolver: zodResolver(productSchema),
  defaultValues: { name: '', costPrice: 0 },
});

return (
  <Form {...form}>
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <FormField
        control={form.control}
        name="name"
        render={({ field }) => (
          <FormItem>
            <FormLabel>Product Name</FormLabel>
            <FormControl>
              <Input placeholder="Enter product name" {...field} />
            </FormControl>
            <FormMessage />  {/* Auto-displays Zod validation errors */}
          </FormItem>
        )}
      />
    </form>
  </Form>
);
```

For simple forms (toggles, inline edits) that don't need React Hook Form, use basic pattern:

```typescript
<div>
  <Label htmlFor="name">Product Name</Label>
  <Input id="name" value={name} onChange={e => setName(e.target.value)} />
  {error && <p className="text-sm text-destructive mt-1">{error}</p>}
</div>
```

### localStorage Errors

Silently fall back to defaults. Never crash the app because localStorage is full or unavailable.

---

## Accessibility Standards

- All interactive elements are keyboard-accessible.
- All form inputs have associated `<Label>` elements (via `htmlFor` or shadcn `FormLabel`).
- All images have `alt` text.
- Color is never the sole indicator — always pair with text or icons.
- Use `aria-live="polite"` for dynamic content updates (toast notifications, alert counts).
- Focus management: return focus to trigger element when dialogs close.
- Table sort headers use `aria-sort="ascending"` / `"descending"` / `"none"`.
- Checkbox indeterminate state uses `aria-checked="mixed"`.
- Clickable stat cards use `role="button"` and `tabIndex={0}`.
- shadcn/ui components handle internal ARIA attributes (Dialog, Accordion, etc.). You may customize the theme (colors, sizes) but do not override their ARIA behavior. Add `aria-label` to custom wrapper components where needed.
