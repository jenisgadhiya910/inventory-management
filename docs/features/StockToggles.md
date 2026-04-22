# Stock Toggles (In-Stock, Featured, Settings)

## What It Does

Toggles are simple on/off switches used throughout the app. They let users mark products as in-stock or out-of-stock, highlight featured items, and manage system settings. Every toggle saves immediately.

---

## Component Architecture

```
components/shared/
├── ToggleSwitch.tsx              # Reusable wrapper around shadcn Switch with label + state

features/products/components/
├── StockStatusToggle.tsx         # In-stock/out-of-stock toggle with side effects
├── FeaturedToggle.tsx            # Featured item star toggle

features/settings/components/
├── ThemeToggle.tsx               # Dark mode switch with sun/moon icons
├── AlertSettingsToggles.tsx      # Per-alert-type notification toggles

features/products/components/
├── FilterToggles.tsx             # Quick filter toggle buttons (low stock, featured, archived)
```

---

## Toggle Types & Implementation

### 1. In-Stock / Out-of-Stock Toggle

**Locations:** Product table rows, product detail panel, edit product dialog

```typescript
interface StockStatusToggleProps {
  product: Product;
  onToggle: (productId: string, isInStock: boolean) => void;
}
```

**Toggle On (In Stock):**
- Set `product.isInStock = true`
- Status badge -> green "In Stock"
- Product row returns to normal opacity

**Toggle Off (Out of Stock):**
- Set `product.isInStock = false`, `product.quantity = 0`
- Status badge -> red "Out of Stock"
- Product row gets `opacity-50` class
- Create StockAlert `{ type: 'out_of_stock', severity: 'critical' }`
- Create StockTransaction `{ type: 'adjustment', previousQuantity: old, newQuantity: 0 }`
- Show toast with "Undo" action (5-second duration)

**Undo Logic:**
```typescript
toast('Product marked as out of stock', {
  action: {
    label: 'Undo',
    onClick: () => {
      updateProduct({ ...product, isInStock: true, quantity: previousQuantity });
      clearAlert(alertId);
    },
  },
  duration: 5000,
});
```

**Auto-sync:** When quantity reaches 0 through other means (inline edit, sale transaction), `isInStock` is automatically set to `false`.

---

### 2. Featured Item Toggle

**Locations:** Product detail panel, edit product dialog, product table

```typescript
interface FeaturedToggleProps {
  productId: string;
  isFeatured: boolean;
  onToggle: (productId: string, isFeatured: boolean) => void;
}
```

- **On**: Star icon filled, product sortable by "Featured first"
- **Off**: Star icon outlined, regular product
- No side effects beyond updating the product's `isFeatured` field
- Sort option: `products.sort((a, b) => Number(b.isFeatured) - Number(a.isFeatured))`

---

### 3. Supplier Active Toggle

**Locations:** Supplier detail panel, supplier table

```typescript
interface SupplierActiveToggleProps {
  supplierId: string;
  isActive: boolean;
  onToggle: (supplierId: string, isActive: boolean) => void;
}
```

- **On**: Supplier available in product form dropdowns
- **Off**: Supplier hidden from dropdowns, but existing product links preserved
- Filter supplier list: `suppliers.filter(s => s.isActive)` for form selects

---

### 4. Dark Mode Toggle

**Location:** App header menu, settings page

```typescript
// features/settings/hooks/useTheme.ts
type Theme = 'light' | 'dark' | 'system';

function useTheme() {
  const [theme, setTheme] = useLocalStorage<Theme>('stockbase_settings.theme', 'system');

  useEffect(() => {
    const root = document.documentElement;
    const resolved = theme === 'system'
      ? (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light')
      : theme;
    root.classList.toggle('dark', resolved === 'dark');
  }, [theme]);

  return { theme, setTheme };
}
```

- Toggle between sun icon (light) and moon icon (dark)
- Changes apply instantly via Tailwind `dark:` variants
- Persisted in `stockbase_settings.theme`

---

### 5. Alert Notification Toggles

**Location:** Settings page

```typescript
interface AlertSettings {
  low_stock: boolean;
  out_of_stock: boolean;
  overstock: boolean;
  reorder: boolean;
}
```

Each alert type has its own `Switch`:
- **Low Stock Alerts**: Suppress/enable `low_stock` alert creation
- **Out of Stock Alerts**: Suppress/enable `out_of_stock` alert creation
- **Overstock Alerts**: Suppress/enable `overstock` alert creation
- **Reorder Reminders**: Suppress/enable `reorder` alert creation

When disabled, the alert engine skips creating alerts of that type.

---

## Filter Toggles

**Location:** Product table filter bar

Quick-toggle buttons (not switches — these are `Toggle` components):

```typescript
<div className="flex gap-2">
  <Toggle pressed={showLowStockOnly} onPressedChange={setShowLowStockOnly}>
    Low Stock Only
  </Toggle>
  <Toggle pressed={showFeaturedOnly} onPressedChange={setShowFeaturedOnly}>
    Featured Only
  </Toggle>
  <Toggle pressed={showArchived} onPressedChange={setShowArchived}>
    Show Archived
  </Toggle>
</div>
```

- These integrate with `useProductFilters` — they set filter state, not product state
- Visually: lit up (primary color background) when active, outlined when inactive

---

## Persistence

| Toggle | Persisted | Storage |
|--------|-----------|---------|
| In-stock / Out-of-stock | Yes | `stockbase_products` (product.isInStock) |
| Featured item | Yes | `stockbase_products` (product.isFeatured) |
| Supplier active | Yes | `stockbase_suppliers` (supplier.isActive) |
| Dark mode | Yes | `stockbase_settings` (settings.theme) |
| Alert notifications | Yes | `stockbase_settings` (settings.alertsEnabled) |
| Filter toggles | No | React state only, resets on navigation |

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Toggle stock off then undo after 5s | Undo expired, product stays out of stock |
| Toggle stock off for product with active orders | Not applicable (no order system), just toggle |
| Disable all alert types | No alerts created, existing alerts remain visible |
| Toggle featured on 100+ products | No limit, all get isFeatured = true |
| Dark mode + system preference change | If set to "system", react to `matchMedia` change event |

---

## Accessibility

- All toggles operable via keyboard (Space or Enter)
- Each toggle has a visible `<Label>` with `htmlFor` linking
- Screen readers announce current state: "In Stock, switch, on" / "In Stock, switch, off"
- Focus outlines visible in both light and dark mode
- `aria-live="polite"` on toast notifications for toggle feedback

---

## Test Scenarios

| Test | Type |
|------|------|
| Stock toggle changes product.isInStock | Unit |
| Stock toggle off creates out_of_stock alert | Integration |
| Undo on stock toggle restores previous state | Component |
| Featured toggle updates product.isFeatured | Unit |
| Dark mode toggle applies dark class to document | Component |
| Alert settings toggle suppresses alert creation | Integration |
| Filter toggles update product list | Component |
| Toggle state persists across page reload | E2E |
