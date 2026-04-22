# Architecture & Technical Design

## Overview

StockBase is a client-side-only React application. All data lives in `localStorage`; there is no backend, API, or database. The architecture is designed around feature-based folder structure, React Context for state management, and custom hooks for data access.

---

## Build & Configuration

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### TypeScript Configuration

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",            // React 19 automatic JSX transform — do NOT import React
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"]
}
```

### shadcn/ui Configuration

```jsonc
// components.json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/styles/globals.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "hooks": "@/hooks"
  }
}
```

**Adding components:** `npx shadcn@latest add <component-name>` (e.g., `npx shadcn@latest add button dialog table`).

### Tailwind CSS v4

Tailwind v4 uses CSS-first configuration. Global styles in `src/styles/globals.css`:

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.205 0.017 265.755);
  --color-destructive: oklch(0.577 0.245 27.325);
  /* shadcn/ui theme variables are auto-configured */
}

.dark {
  --color-primary: oklch(0.922 0.016 265.755);
  /* Dark mode overrides */
}
```

### Package Dependencies

```jsonc
// Key dependencies (install with npm)
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "react-router-dom": "^7.0.0",
    "react-hook-form": "^7.0.0",
    "@hookform/resolvers": "^3.0.0",
    "zod": "^3.0.0",
    "recharts": "^2.0.0",
    "date-fns": "^3.0.0",
    "lucide-react": "latest",
    "sonner": "latest",
    "class-variance-authority": "latest",
    "clsx": "latest",
    "tailwind-merge": "latest"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "tailwindcss": "^4.0.0",
    "vitest": "^2.0.0",
    "@testing-library/react": "^16.0.0",
    "@testing-library/user-event": "^14.0.0",
    "@playwright/test": "^1.0.0",
    "react-error-boundary": "^4.0.0"
  }
}
```

---

## Folder Structure

```
src/
├── app/                        # App entry, routing, providers
│   ├── App.tsx                 # Root component with RouterProvider
│   ├── routes.tsx              # Route definitions (react-router-dom v7)
│   └── providers.tsx           # Composed context providers
│
├── components/
│   ├── ui/                     # shadcn/ui primitives (Button, Input, Dialog, etc.)
│   └── shared/                 # Reusable non-primitive components
│       ├── PageHeader.tsx
│       ├── EmptyState.tsx
│       ├── ConfirmDialog.tsx
│       └── StatusBadge.tsx
│
├── features/                   # Feature modules (domain-driven)
│   ├── products/
│   │   ├── components/         # ProductTable, ProductRow, InlineEdit, etc.
│   │   ├── hooks/              # useProducts, useProductFilters, useProductForm
│   │   ├── context/            # ProductContext, ProductProvider
│   │   ├── utils/              # Product calculations, validators
│   │   └── types.ts            # Product-specific types (if not in shared)
│   │
│   ├── wizard/
│   │   ├── components/         # WizardShell, StepIndicator, Step1Form, Step2Form, Step3Form
│   │   ├── hooks/              # useWizardForm, useWizardNavigation
│   │   └── validation.ts       # Zod schemas per step
│   │
│   ├── categories/
│   │   ├── components/         # CategoryAccordion, CategoryHeader, CategoryActions
│   │   ├── hooks/              # useCategories, useCategoryTree
│   │   └── context/            # CategoryContext
│   │
│   ├── suppliers/
│   │   ├── components/         # SupplierTable, SupplierForm
│   │   ├── hooks/              # useSuppliers
│   │   └── context/            # SupplierContext
│   │
│   ├── alerts/
│   │   ├── components/         # AlertList, AlertCard, AlertBanner
│   │   ├── hooks/              # useAlerts, useAlertEngine
│   │   └── context/            # AlertContext
│   │
│   ├── analytics/
│   │   ├── components/         # Each chart as its own component
│   │   ├── hooks/              # useChartData, useDateRange
│   │   └── utils/              # Aggregation, calculations, forecasting
│   │
│   └── settings/
│       ├── components/         # ThemeToggle, AlertSettings
│       └── hooks/              # useSettings
│
├── hooks/                      # App-wide custom hooks
│   ├── useLocalStorage.ts      # Generic localStorage read/write with JSON parse
│   ├── useDebounce.ts          # Debounce for search inputs
│   └── useMediaQuery.ts        # Responsive breakpoint detection
│
├── lib/                        # Utilities and helpers
│   ├── storage.ts              # localStorage wrapper with key constants
│   ├── id.ts                   # ID generation (crypto.randomUUID or nanoid)
│   ├── format.ts               # Currency, date, percentage formatters
│   └── constants.ts            # App-wide constants (default categories, units, etc.)
│
├── types/                      # Shared TypeScript types/interfaces
│   └── index.ts                # Product, Category, Supplier, StockAlert, StockTransaction
│
└── styles/                     # Global styles and Tailwind config overrides
    └── globals.css
```

### Key Principles

- **Feature-based organization**: Each domain (products, categories, suppliers, alerts, analytics) is self-contained with its own components, hooks, context, and utils.
- **No cross-feature imports of components**: Features communicate through shared types and context. If `ProductTable` needs category data, it reads from `CategoryContext`, not from `features/categories/components/`. Cross-feature imports of **types** and **hooks** are allowed.
- **`components/ui/`** is reserved for shadcn/ui primitives only. Do not put business logic here. These are installed via `npx shadcn@latest add <component>` and should not be manually modified.
- **`components/shared/`** contains reusable composed components that span features (e.g., `ConfirmDialog`, `EmptyState`).
- **`@/` path alias** maps to `src/`. All imports use `@/` prefix: `import { Button } from '@/components/ui/button'`.

---

## State Management

### Architecture: React Context + useReducer

Each domain has its own Context + Provider that wraps the relevant routes. All providers are composed in `src/app/providers.tsx`.

```typescript
// Pattern for each feature context
interface ProductState {
  products: Product[];
  loading: boolean;
  filters: ProductFilters;
}

type ProductAction =
  | { type: 'SET_PRODUCTS'; payload: Product[] }
  | { type: 'ADD_PRODUCT'; payload: Product }
  | { type: 'UPDATE_PRODUCT'; payload: Product }
  | { type: 'DELETE_PRODUCT'; payload: string }
  | { type: 'SET_FILTERS'; payload: ProductFilters };

function productReducer(state: ProductState, action: ProductAction): ProductState {
  // ... reducer logic
}
```

### Provider Composition Order

```typescript
// src/app/providers.tsx
export function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider>           {/* Outermost: theme affects all UI */}
      <SettingsProvider>      {/* Settings needed by alert engine */}
        <CategoryProvider>    {/* Categories referenced by products */}
          <SupplierProvider>  {/* Suppliers referenced by products */}
            <ProductProvider> {/* Products referenced by alerts */}
              <AlertProvider> {/* Innermost: alerts read from products + settings */}
                {children}
              </AlertProvider>
            </ProductProvider>
          </SupplierProvider>
        </CategoryProvider>
      </SettingsProvider>
    </ThemeProvider>
  );
}
```

**Why this order?** Inner providers can read from outer providers via hooks. AlertProvider is innermost because it needs to read ProductContext (to evaluate stock levels) and SettingsContext (to check which alert types are enabled). Products reference Categories and Suppliers by ID.

**ThemeProvider** is a custom provider (not `next-themes`). It reads `stockbase_settings.theme` and toggles the `dark` class on `document.documentElement`:

```typescript
// features/settings/context/ThemeProvider.tsx
type Theme = 'light' | 'dark' | 'system';

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>(() =>
    getStorageItem<Theme>('stockbase_settings.theme', 'system')
  );

  useEffect(() => {
    const resolved = theme === 'system'
      ? (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light')
      : theme;
    document.documentElement.classList.toggle('dark', resolved === 'dark');
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### State-to-localStorage Sync

Each provider syncs its entity list to localStorage when items change:

```typescript
// Inside each provider — sync ONLY the entity array, not the full state
const prevItemsRef = useRef(state.items);

useEffect(() => {
  // Only write if items actually changed (avoid writes on filter/loading changes)
  if (prevItemsRef.current !== state.items) {
    prevItemsRef.current = state.items;
    setStorageItem(STORAGE_KEY, state.items);
  }
}, [state.items]);
```

**Initial load**: Read from localStorage in the useReducer lazy initializer (third argument). This avoids a flash of empty state:

```typescript
const [state, dispatch] = useReducer(productReducer, null, () => {
  const stored = getStorageItem<Product[]>(STORAGE_KEYS.PRODUCTS, []);
  return {
    products: stored,
    loading: false,    // loading is always false for localStorage (synchronous)
    filters: defaultFilters,
  };
});
```

> **Important:** The third argument to `useReducer` is required for lazy initialization. Without it, the initializer runs on every render. The `loading` field exists for future API migration but is always `false` in the localStorage version.

---

## Data Flow Patterns

### Reading Data

```
Component → useProducts() hook → ProductContext → state.products
```

Hooks expose filtered/computed data. Components never read raw context directly.

```typescript
// features/products/hooks/useProducts.ts
export function useProducts() {
  const { state, dispatch } = useContext(ProductContext);

  const addProduct = (product: Omit<Product, 'id' | 'createdAt' | 'updatedAt'>) => {
    const newProduct: Product = {
      ...product,
      id: generateId(),
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };
    dispatch({ type: 'ADD_PRODUCT', payload: newProduct });
  };

  return { products: state.products, addProduct, /* ... */ };
}
```

### Cross-Feature Communication

When an action in one feature affects another (e.g., restocking a product should clear its alert):

```
ProductProvider.restockProduct()
  -> updates product quantity
  -> calls alertEngine.evaluateProduct(product)
    -> AlertProvider.clearAlert() or AlertProvider.createAlert()
```

The `useAlertEngine` hook (located in `features/alerts/hooks/useAlertEngine.ts`) is the bridge between features. It reads from `ProductContext` and `SettingsContext`, and writes to `AlertContext`. Any component that mutates product quantities must call `alertEngine.evaluateProduct(product)` after the mutation.

```typescript
// features/alerts/hooks/useAlertEngine.ts
export function useAlertEngine() {
  const { alerts, addAlert, removeAlert } = useAlerts();
  const { settings } = useSettings();

  const evaluateProduct = (product: Product, supplier?: Supplier) => {
    // Check each alert type config, create or clear alerts as needed
    // See StockAlerts.md for full implementation
  };

  const clearAlertsForProduct = (productId: string) => {
    alerts.filter(a => a.productId === productId).forEach(a => removeAlert(a.id));
  };

  return { evaluateProduct, clearAlertsForProduct };
}
```

**Rule:** Do not have providers dispatch actions to each other directly. Always go through hooks.

### Data Mutation Flow

```
User Action → Component handler → hook.mutate() → dispatch(action) → reducer → new state → useEffect syncs to localStorage
```

Every mutation goes through the reducer. No direct `localStorage.setItem` calls outside of the sync effect.

---

## Routing

Using `react-router-dom` v7 with `createBrowserRouter`:

```typescript
// src/app/routes.tsx
import { createBrowserRouter, Navigate } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <AppLayout />,
    errorElement: <AppErrorPage />,
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'products', element: <ProductListing /> },
      { path: 'products/new', element: <AddProductWizard /> },
      { path: 'products/:id', element: <ProductDetail /> },
      { path: 'categories', element: <CategoryManagement /> },
      { path: 'suppliers', element: <SupplierManagement /> },
      { path: 'alerts', element: <AlertsPage /> },
      { path: 'analytics', element: <AnalyticsPage /> },
      { path: 'settings', element: <SettingsPage /> },
      { path: '*', element: <Navigate to="/" replace /> },
    ],
  },
]);
```

### App Entry Point

```typescript
// src/app/App.tsx
import { RouterProvider } from 'react-router-dom';
import { ErrorBoundary } from 'react-error-boundary';

export function App() {
  return (
    <ErrorBoundary fallback={<AppCrashScreen />}>
      <AppProviders>
        <RouterProvider router={router} />
      </AppProviders>
    </ErrorBoundary>
  );
}
```

### AppLayout Component

```typescript
// src/components/shared/AppLayout.tsx
import { Outlet } from 'react-router-dom';

export function AppLayout() {
  return (
    <div className="flex h-screen">
      {/* Desktop sidebar — hidden on mobile */}
      <aside className="hidden md:flex md:w-64 md:flex-col md:border-r bg-background">
        <div className="p-4 border-b">
          <h1 className="text-lg font-bold">StockBase</h1>
        </div>
        <nav className="flex-1 p-2 space-y-1">
          <NavLink to="/" icon={LayoutDashboard} label="Dashboard" />
          <NavLink to="/products" icon={Package} label="Products" badge={productCount} />
          <NavLink to="/categories" icon={FolderTree} label="Categories" badge={categoryCount} />
          <NavLink to="/suppliers" icon={Truck} label="Suppliers" badge={supplierCount} />
          <NavLink to="/alerts" icon={Bell} label="Alerts" badge={unreadAlertCount} badgeVariant="critical" />
          <NavLink to="/analytics" icon={BarChart3} label="Analytics" />
          <NavLink to="/settings" icon={Settings} label="Settings" />
        </nav>
      </aside>

      {/* Main content area */}
      <main className="flex-1 overflow-auto">
        <Outlet />
      </main>

      {/* Mobile bottom tab bar — visible only on mobile */}
      <nav className="fixed bottom-0 left-0 right-0 flex justify-around border-t bg-background p-2 md:hidden">
        <MobileTabLink to="/" icon={LayoutDashboard} />
        <MobileTabLink to="/products" icon={Package} />
        <MobileTabLink to="/categories" icon={FolderTree} />
        <MobileTabLink to="/alerts" icon={Bell} badge={unreadAlertCount} />
        <MobileTabLink to="/settings" icon={Settings} />
      </nav>
    </div>
  );
}
```

### URL-Driven Navigation

Tabs map to routes (not local state). This enables deep linking, bookmarking, and browser back/forward:

- `/products?status=low_stock` — ProductTable reads `status` query param and applies filter
- `/products?status=out_of_stock` — Same pattern for out-of-stock filter
- `/products/new?categoryId=cat-1` — AddProductWizard reads `categoryId` and pre-fills category select

Query param reading pattern:
```typescript
const [searchParams] = useSearchParams();
const statusFilter = searchParams.get('status') as StockStatus | null;
const prefilledCategoryId = searchParams.get('categoryId');
```

### Product Detail: Route vs. Dialog

- `/products/:id` is a **full route** (separate page with back navigation)
- "Quick view" from the product table opens a **Dialog overlay** (no URL change)
- Edit and Restock are always **Dialog overlays** opened from either the detail page or the table

---

## localStorage Strategy

### Key Registry

All keys are defined in `src/lib/storage.ts`:

```typescript
export const STORAGE_KEYS = {
  PRODUCTS: 'stockbase_products',
  CATEGORIES: 'stockbase_categories',
  SUPPLIERS: 'stockbase_suppliers',
  ALERTS: 'stockbase_alerts',
  TRANSACTIONS: 'stockbase_transactions',
  SETTINGS: 'stockbase_settings',
} as const;
```

### Storage Wrapper

```typescript
export function getStorageItem<T>(key: string, fallback: T): T {
  try {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : fallback;
  } catch {
    return fallback;
  }
}

export function setStorageItem<T>(key: string, value: T): void {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch (e) {
    console.error(`Failed to write to localStorage key "${key}":`, e);
  }
}
```

### Storage Limits

localStorage has a ~5MB limit. For this app, this is sufficient. If data grows large:
- Products with base64 images can bloat quickly. Store image URLs or use `URL.createObjectURL` for previews only.
- Stock transactions should be capped (keep last 1000 per product).

---

## Form Validation

Use **Zod** for schema validation, integrated with React Hook Form where multi-field forms are involved.

```typescript
// features/wizard/validation.ts
import { z } from 'zod';

export const step1Schema = z.object({
  name: z.string().min(3, 'Product name must be at least 3 characters').max(100),
  description: z.string().max(500).optional(),
  sku: z.string().optional(),
  categoryId: z.string().min(1, 'Please select a category'),
  supplierId: z.string().min(1, 'Please select a supplier'),
  tags: z.array(z.string()),
});

export const step2Schema = z.object({
  costPrice: z.number().positive('Cost price must be greater than 0'),
  sellingPrice: z.number().positive('Selling price must be greater than 0'),
}).refine(data => data.sellingPrice >= data.costPrice, {
  message: 'Selling price cannot be less than cost price',
  path: ['sellingPrice'],
});

export const step3Schema = z.object({
  quantity: z.number().min(0, 'Quantity cannot be negative'),
  unit: z.enum(['pieces', 'kg', 'liters', 'meters', 'boxes']),
  minStockLevel: z.number().min(0, 'Minimum stock level is required'),
  maxStockLevel: z.number().optional(),
  location: z.string().optional(),
});
```

---

## ID Generation

Use `crypto.randomUUID()` for all entity IDs. It is available in all modern browsers and produces unique, collision-resistant IDs without external dependencies.

```typescript
// src/lib/id.ts
export function generateId(): string {
  return crypto.randomUUID();
}
```

---

## Error Boundaries

Use `react-error-boundary` package (v4+) for error boundaries:

```typescript
import { ErrorBoundary } from 'react-error-boundary';

// App-level: catches routing or provider failures
<ErrorBoundary fallback={<AppCrashScreen />}>
  <AppProviders>
    <RouterProvider router={router} />
  </AppProviders>
</ErrorBoundary>

// Feature-level: catches chart rendering failures without taking down the page
<ErrorBoundary fallback={<ChartErrorFallback />}>
  <StockTrendChart />
</ErrorBoundary>
```

**AppCrashScreen**: Full-page message with "Reload" button and option to clear localStorage.
**ChartErrorFallback**: Card with "Failed to load chart" message and "Retry" button.

---

## Performance Considerations

| Concern | Strategy |
|---------|----------|
| Large product lists (1000+) | Use `react-window` for virtualized table rows |
| Expensive chart calculations | Memoize with `useMemo`, recompute only when source data or filters change |
| Search/filter debouncing | `useDebounce(searchTerm, 300)` before filtering |
| Re-renders from context | Split contexts per feature; use `React.memo` on row components |
| localStorage reads | Read once on mount (lazy initializer), never on every render |
| Image handling | Store as URLs only. For local file uploads, use `URL.createObjectURL()` for preview display during the session but do NOT persist blob URLs to localStorage (they expire). Instead, store images as base64 data URIs only as a last resort, keeping them under 200KB each. In production, images would be uploaded to a CDN. |
