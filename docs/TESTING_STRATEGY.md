# Testing Strategy

## Overview

StockBase uses a three-layer testing approach: unit tests for logic, component tests for UI behavior, and end-to-end tests for critical user flows.

---

## Test Stack

| Tool | Purpose |
|------|---------|
| **Vitest** | Test runner and assertion library (compatible with Vite) |
| **React Testing Library** | Component rendering and DOM interaction |
| **@testing-library/user-event** | Simulating realistic user interactions |
| **MSW (Mock Service Worker)** | Not needed (no API), but useful if external integrations are added later |
| **Playwright** | End-to-end browser tests for critical flows |

---

## Test File Location

Tests are co-located with the files they test:

```
features/products/hooks/useProducts.ts
features/products/hooks/useProducts.test.ts
features/products/components/ProductTable.tsx
features/products/components/ProductTable.test.tsx
```

---

## Layer 1: Unit Tests

### What to Unit Test

- **Reducers**: Every action type produces the correct state.
- **Utility functions**: Formatters, calculators, validators, ID generators.
- **Zod schemas**: Validation accepts valid input, rejects invalid input with correct error messages.
- **Pure computation hooks**: Hooks that transform data (filtering, sorting, aggregation).

### Examples

```typescript
// features/products/utils/calculations.test.ts
describe('calculateProfitMargin', () => {
  it('returns correct margin for normal prices', () => {
    expect(calculateProfitMargin(80, 100)).toBe(20); // 20%
  });

  it('returns 0 when cost equals selling price', () => {
    expect(calculateProfitMargin(100, 100)).toBe(0);
  });

  it('returns negative margin when selling below cost', () => {
    expect(calculateProfitMargin(100, 80)).toBe(-20);
  });

  it('handles zero cost price without dividing by zero', () => {
    expect(calculateProfitMargin(0, 50)).toBe(100);
  });
});
```

```typescript
// features/wizard/validation.test.ts
describe('step1Schema', () => {
  it('rejects name shorter than 3 characters', () => {
    const result = step1Schema.safeParse({ name: 'AB', categoryId: '1', supplierId: '1', tags: [] });
    expect(result.success).toBe(false);
    expect(result.error?.issues[0].message).toBe('Product name must be at least 3 characters');
  });

  it('accepts valid step 1 data', () => {
    const result = step1Schema.safeParse({
      name: 'Test Product',
      categoryId: 'cat-1',
      supplierId: 'sup-1',
      tags: ['fragile'],
    });
    expect(result.success).toBe(true);
  });
});
```

```typescript
// features/alerts/hooks/useAlertEngine.test.ts
describe('evaluateProduct', () => {
  it('creates low_stock alert when quantity <= minStockLevel and > 0', () => {
    const product = createMockProduct({ quantity: 5, minStockLevel: 10 });
    const alerts = evaluateProduct(product);
    expect(alerts).toContainEqual(expect.objectContaining({ type: 'low_stock', severity: 'warning' }));
  });

  it('creates out_of_stock alert when quantity is 0', () => {
    const product = createMockProduct({ quantity: 0 });
    const alerts = evaluateProduct(product);
    expect(alerts).toContainEqual(expect.objectContaining({ type: 'out_of_stock', severity: 'critical' }));
  });

  it('creates overstock alert when quantity > maxStockLevel', () => {
    const product = createMockProduct({ quantity: 150, maxStockLevel: 100 });
    const alerts = evaluateProduct(product);
    expect(alerts).toContainEqual(expect.objectContaining({ type: 'overstock', severity: 'info' }));
  });

  it('returns no alerts for healthy stock levels', () => {
    const product = createMockProduct({ quantity: 50, minStockLevel: 10, maxStockLevel: 100 });
    const alerts = evaluateProduct(product);
    expect(alerts).toHaveLength(0);
  });
});
```

---

## Layer 2: Component Tests

### What to Component Test

- **User interactions**: Click, type, select, toggle, navigate.
- **Conditional rendering**: Empty states, loading states, error states.
- **Form validation**: Inline errors appear on invalid input, disappear on correction.
- **Accessibility**: Elements are labelled, focusable, keyboard-operable.

### Testing Pattern

```typescript
// features/products/components/ProductTable.test.tsx
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ProductTable } from './ProductTable';
import { TestProviders } from '@/test/helpers';

const mockProducts = [
  createMockProduct({ name: 'Widget A', quantity: 50, minStockLevel: 10 }),
  createMockProduct({ name: 'Widget B', quantity: 3, minStockLevel: 10 }),
  createMockProduct({ name: 'Widget C', quantity: 0 }),
];

describe('ProductTable', () => {
  it('renders all products', () => {
    render(<ProductTable />, { wrapper: createTestProviders({ products: mockProducts }) });
    expect(screen.getByText('Widget A')).toBeInTheDocument();
    expect(screen.getByText('Widget B')).toBeInTheDocument();
    expect(screen.getByText('Widget C')).toBeInTheDocument();
  });

  it('shows empty state when no products exist', () => {
    render(<ProductTable />, { wrapper: createTestProviders({ products: [] }) });
    expect(screen.getByText(/no products yet/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /add.*product/i })).toBeInTheDocument();
  });

  it('filters products by search term', async () => {
    const user = userEvent.setup();
    render(<ProductTable />, { wrapper: createTestProviders({ products: mockProducts }) });

    await user.type(screen.getByPlaceholderText(/search/i), 'Widget A');
    expect(screen.getByText('Widget A')).toBeInTheDocument();
    expect(screen.queryByText('Widget B')).not.toBeInTheDocument();
  });

  it('shows correct stock status badges', () => {
    render(<ProductTable />, { wrapper: createTestProviders({ products: mockProducts }) });
    expect(screen.getByText('In Stock')).toBeInTheDocument();
    expect(screen.getByText('Low Stock')).toBeInTheDocument();
    expect(screen.getByText('Out of Stock')).toBeInTheDocument();
  });

  it('enables bulk actions when products are selected', async () => {
    const user = userEvent.setup();
    render(<ProductTable />, { wrapper: createTestProviders({ products: mockProducts }) });

    const checkboxes = screen.getAllByRole('checkbox');
    await user.click(checkboxes[1]); // first product checkbox

    expect(screen.getByRole('button', { name: /delete/i })).toBeInTheDocument();
  });
});
```

### localStorage Mocking

```typescript
// test/helpers.ts
export function mockLocalStorage(data: Record<string, unknown> = {}) {
  const store: Record<string, string> = {};
  for (const [key, value] of Object.entries(data)) {
    store[key] = JSON.stringify(value);
  }

  vi.spyOn(Storage.prototype, 'getItem').mockImplementation((key) => store[key] ?? null);
  vi.spyOn(Storage.prototype, 'setItem').mockImplementation((key, value) => { store[key] = value; });
  vi.spyOn(Storage.prototype, 'removeItem').mockImplementation((key) => { delete store[key]; });

  return store;
}
```

### Test Providers Wrapper

```typescript
// test/helpers.ts
export function createTestProviders(initialData: Partial<AppState> = {}) {
  return function TestProviders({ children }: { children: React.ReactNode }) {
    return (
      <MemoryRouter>
        <AppProviders initialState={initialData}>
          {children}
        </AppProviders>
      </MemoryRouter>
    );
  };
}
```

---

## Layer 3: End-to-End Tests (Playwright)

### Critical User Flows to Test

| Flow | Steps |
|------|-------|
| **Add Product** | Navigate to wizard -> fill Step 1 -> fill Step 2 -> fill Step 3 -> submit -> verify product appears in table |
| **Restock Product** | Open product -> click Restock -> enter quantity -> confirm -> verify quantity updated + alert cleared |
| **Delete Product** | Select product -> click Delete -> confirm -> verify product removed from table |
| **Filter & Search** | Type in search -> verify results filtered -> apply category filter -> verify combined filtering |
| **Stock Alert Lifecycle** | Add product with low stock -> verify alert created -> restock product -> verify alert cleared |
| **Dark Mode Toggle** | Toggle dark mode -> verify theme changes -> reload page -> verify theme persists |
| **Category Management** | Create category -> add product to it -> expand accordion -> verify product visible -> delete category with reassignment |

### E2E Test Structure

```typescript
// e2e/add-product.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Add Product Wizard', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
    // Clear localStorage for clean state
    await page.evaluate(() => localStorage.clear());
    await page.goto('/products/new');
  });

  test('completes full product creation flow', async ({ page }) => {
    // Step 1
    await page.getByLabel('Product Name').fill('Test Widget');
    await page.getByLabel('Category').click();
    await page.getByRole('option', { name: 'Electronics' }).click();
    await page.getByLabel('Supplier').click();
    await page.getByRole('option').first().click();
    await page.getByRole('button', { name: 'Next' }).click();

    // Step 2
    await page.getByLabel('Cost Price').fill('80');
    await page.getByLabel('Selling Price').fill('100');
    await page.getByRole('button', { name: 'Next' }).click();

    // Step 3
    await page.getByLabel('Initial Quantity').fill('50');
    await page.getByLabel('Minimum Stock Level').fill('10');
    await page.getByRole('button', { name: 'Add Product' }).click();

    // Verify
    await expect(page).toHaveURL('/products');
    await expect(page.getByText('Test Widget')).toBeVisible();
    await expect(page.getByText('Product added successfully')).toBeVisible();
  });

  test('shows validation errors for empty required fields', async ({ page }) => {
    await page.getByRole('button', { name: 'Next' }).click();
    await expect(page.getByText('Product name is required')).toBeVisible();
    await expect(page.getByText('Please select a category')).toBeVisible();
  });
});
```

---

## Test Data Factories

Create reusable mock data builders:

```typescript
// test/factories.ts
import type { Product, Category, Supplier, StockAlert } from '@/types';
import { generateId } from '@/lib/id';

export function createMockProduct(overrides: Partial<Product> = {}): Product {
  return {
    id: generateId(),
    name: 'Test Product',
    description: '',
    sku: `SKU-${Date.now()}`,
    barcode: '',
    categoryId: 'cat-1',
    supplierId: 'sup-1',
    costPrice: 80,
    sellingPrice: 100,
    quantity: 50,
    minStockLevel: 10,
    maxStockLevel: 100,
    unit: 'pieces',
    images: [],
    isInStock: true,
    isFeatured: false,
    isArchived: false,
    tags: [],
    location: '',
    weight: null,
    dimensions: null,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    ...overrides,
  };
}

export function createMockCategory(overrides: Partial<Category> = {}): Category {
  return {
    id: generateId(),
    name: 'Test Category',
    description: '',
    parentId: null,
    color: '#3b82f6',
    productCount: 0,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    ...overrides,
  };
}

export function createMockSupplier(overrides: Partial<Supplier> = {}): Supplier {
  return {
    id: generateId(),
    name: 'Test Supplier',
    email: 'test@supplier.com',
    phone: '+1234567890',
    company: 'Test Co',
    address: '123 Test St',
    leadTimeDays: 5,
    rating: 4,
    isActive: true,
    productsSupplied: [],
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    ...overrides,
  };
}
```

---

## Development Fixtures

For manual testing and UI development, use a fixture loader that populates localStorage with realistic data:

```typescript
// test/fixtures.ts — import and call loadDevFixtures() from browser console or a dev-only button
import { STORAGE_KEYS } from '@/lib/storage';
import { createMockProduct, createMockCategory, createMockSupplier } from './factories';

export function loadDevFixtures() {
  const categories = [
    createMockCategory({ id: 'cat-1', name: 'Electronics', color: '#3b82f6', productCount: 8 }),
    createMockCategory({ id: 'cat-2', name: 'Clothing', color: '#8b5cf6', productCount: 5 }),
    createMockCategory({ id: 'cat-3', name: 'Food & Beverages', color: '#22c55e', productCount: 4 }),
    createMockCategory({ id: 'cat-4', name: 'Office Supplies', color: '#f59e0b', productCount: 3 }),
    createMockCategory({ id: 'cat-5', name: 'Raw Materials', color: '#6b7280', productCount: 2 }),
  ];

  const suppliers = [
    createMockSupplier({ id: 'sup-1', name: 'John Smith', company: 'TechSupply Co', leadTimeDays: 5, rating: 4 }),
    createMockSupplier({ id: 'sup-2', name: 'Sarah Lee', company: 'FreshGoods Inc', leadTimeDays: 3, rating: 5 }),
    createMockSupplier({ id: 'sup-3', name: 'Mike Chen', company: 'GlobalParts Ltd', leadTimeDays: 7, rating: 3 }),
  ];

  const products = [
    // Mix of healthy, low stock, and out of stock for realistic UI testing
    createMockProduct({ name: 'Wireless Mouse', categoryId: 'cat-1', supplierId: 'sup-1', quantity: 45, minStockLevel: 10, costPrice: 15, sellingPrice: 29.99 }),
    createMockProduct({ name: 'USB-C Cable', categoryId: 'cat-1', supplierId: 'sup-1', quantity: 3, minStockLevel: 20, costPrice: 5, sellingPrice: 12.99 }),  // low stock
    createMockProduct({ name: 'Bluetooth Speaker', categoryId: 'cat-1', supplierId: 'sup-3', quantity: 0, minStockLevel: 5, costPrice: 40, sellingPrice: 79.99, isInStock: false }),  // out of stock
    createMockProduct({ name: 'Cotton T-Shirt', categoryId: 'cat-2', supplierId: 'sup-2', quantity: 120, minStockLevel: 25, maxStockLevel: 100, costPrice: 8, sellingPrice: 24.99 }),  // overstock
    // ... add 15-20 more for realistic table/chart testing
  ];

  localStorage.setItem(STORAGE_KEYS.PRODUCTS, JSON.stringify(products));
  localStorage.setItem(STORAGE_KEYS.CATEGORIES, JSON.stringify(categories));
  localStorage.setItem(STORAGE_KEYS.SUPPLIERS, JSON.stringify(suppliers));
  localStorage.setItem(STORAGE_KEYS.ALERTS, JSON.stringify([]));
  localStorage.setItem(STORAGE_KEYS.TRANSACTIONS, JSON.stringify([]));
}
```

---

## Coverage Targets

| Layer | Target | Focus |
|-------|--------|-------|
| Unit tests | 90%+ | Reducers, utils, validators, calculations |
| Component tests | 80%+ | All user-facing features, all interactive elements |
| E2E tests | 7 critical flows | Add product, restock, delete, search/filter, alerts, settings, categories |

---

## Running Tests

```bash
# Unit + Component tests
npm run test              # Run all tests
npm run test:watch        # Watch mode during development
npm run test:coverage     # Generate coverage report

# E2E tests
npm run test:e2e          # Run Playwright tests
npm run test:e2e:ui       # Run with Playwright UI
```
