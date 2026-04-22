# Stats Cards (Inventory Overview)

## What It Does

Stats cards sit at the top of the dashboard and give a quick snapshot of inventory health. Each card highlights one key metric so users can spot issues at a glance without digging into tables or reports.

> **Dependencies:** Reads from `ProductContext`, `CategoryContext`, `SupplierContext`, and `TransactionContext` (for trend calculations). Clickable cards navigate to [ProductTable](ProductTable.md) with query params (e.g., `/products?status=low_stock`).

---

## Component Architecture

```
features/products/components/
├── StatsCards.tsx               # Grid container rendering all stat cards
├── StatCard.tsx                 # Reusable single stat card component
└── ...

features/products/hooks/
├── useInventoryStats.ts         # Computed stats from products, categories, suppliers, alerts
└── ...
```

### Component Hierarchy

```
<Dashboard>
  <StatsCards>
    <StatCard title="Total Products" value={stats.totalProducts} trend={stats.productsTrend} />
    <StatCard title="Low Stock" value={stats.lowStock} variant="warning" onClick={navigateToLowStock} />
    <StatCard title="Out of Stock" value={stats.outOfStock} variant="critical" onClick={navigateToOutOfStock} />
    <StatCard title="Inventory Value" value={stats.totalValue} format="currency" trend={stats.valueTrend} />
  </StatsCards>
  <SecondaryStats>
    <StatCard title="Categories" value={stats.categoriesCount} />
    <StatCard title="Active Suppliers" value={stats.activeSuppliers} />
    <StatCard title="Items to Reorder" value={stats.itemsToReorder} />
    <StatCard title="Avg. Profit Margin" value={stats.avgMargin} format="percentage" />
  </SecondaryStats>
</Dashboard>
```

---

## StatCard Props Interface

```typescript
interface StatCardProps {
  title: string;
  value: number;
  format?: 'number' | 'currency' | 'percentage';   // default: 'number'
  variant?: 'default' | 'warning' | 'critical';      // controls left border color
  trend?: {
    value: number;        // percentage change
    direction: 'up' | 'down';
  } | null;
  icon?: LucideIcon;
  onClick?: () => void;   // makes card clickable with hover state
}
```

---

## The 8 Cards

### Primary Cards (Top Row)

| # | Title | Calculation | Format | Variant | Clickable |
|---|-------|------------|--------|---------|-----------|
| 1 | Total Products | `products.length` | number | default | No |
| 2 | Low Stock Items | `products.filter(p => p.quantity > 0 && p.quantity <= p.minStockLevel).length` | number | warning | Yes -> `/products?status=low_stock` |
| 3 | Out of Stock | `products.filter(p => p.quantity === 0).length` | number | critical | Yes -> `/products?status=out_of_stock` |
| 4 | Inventory Value | `products.reduce((sum, p) => sum + p.quantity * p.costPrice, 0)` | currency | default | No |

### Secondary Cards (Below)

| # | Title | Calculation | Format |
|---|-------|------------|--------|
| 5 | Categories | `categories.length` | number |
| 6 | Active Suppliers | `suppliers.filter(s => s.isActive).length` | number |
| 7 | Items to Reorder | `products.filter(p => p.quantity > 0 && p.quantity <= p.minStockLevel).length` | number |
| 8 | Avg. Profit Margin | `products.reduce((sum, p) => sum + ((p.sellingPrice - p.costPrice) / p.sellingPrice) * 100, 0) / products.length` | percentage |

---

## Trend Calculation

Trends compare current month to previous month using `StockTransaction` data:

```typescript
function calculateTrend(currentValue: number, previousValue: number): Trend | null {
  if (previousValue === 0) return null; // no trend if no prior data
  const change = ((currentValue - previousValue) / previousValue) * 100;
  return {
    value: Math.abs(Math.round(change)),
    direction: change >= 0 ? 'up' : 'down',
  };
}
```

**Products trend**: Count of products added this month vs. last month.
**Value trend**: Total inventory value end-of-current-month vs. end-of-last-month.

---

## Visual Details

- Each card has: title (muted text, small), large number (font-bold text-2xl), trend indicator (small, below number)
- Trend: green `ArrowUp` icon + percentage = positive; red `ArrowDown` icon + percentage = negative
- Warning variant: `border-l-4 border-amber-500`
- Critical variant: `border-l-4 border-red-500`
- Clickable cards: `cursor-pointer hover:shadow-md transition-shadow`

---

## Responsive Behavior

| Screen Size | Layout |
|---|---|
| Desktop (`lg:`) | 4 primary cards in a row, 4 secondary below |
| Tablet (`md:`) | 2 cards per row |
| Mobile (default) | Single column stack |

```typescript
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4">
  {primaryCards.map(card => <StatCard key={card.title} {...card} />)}
</div>
```

## Empty State

When no products exist, all cards show `0`. Trend indicators are hidden (null). Primary cards still render with their values showing zero. Below the cards, show: "Add your first product to see inventory stats." with a button linking to `/products/new`.

---

## Accessibility

- Each stat card has `role="region"` and `aria-label="{title}: {value}"` (e.g., `aria-label="Low Stock Items: 5"`)
- Clickable cards additionally have `role="button"`, `tabIndex={0}`, and `cursor-pointer`
- Trend indicators have `aria-label="Increased by 12%"` or `aria-label="Decreased by 5%"`
- When trend is null, no indicator renders (no misleading "0%" shown)

---

| Scenario | Behavior |
|----------|----------|
| No products exist | All cards show 0. No trend indicators. |
| Division by zero (avg margin, 0 products) | Return 0, display "0%" |
| No transactions for trend | Trend is `null`, indicator hidden |
| Very large numbers | Use `Intl.NumberFormat` for compact display (e.g., "$1.2M") |
| All products out of stock | Out of Stock card highlighted, "Items to Reorder" shows 0 (nothing to reorder if all at 0) |

---

## useInventoryStats Hook

```typescript
function useInventoryStats() {
  const { products } = useProducts();
  const { categories } = useCategories();
  const { suppliers } = useSuppliers();
  const { transactions } = useTransactions();

  return useMemo(() => ({
    totalProducts: products.length,
    lowStock: products.filter(p => p.quantity > 0 && p.quantity <= p.minStockLevel).length,
    outOfStock: products.filter(p => p.quantity === 0).length,
    totalValue: products.reduce((sum, p) => sum + p.quantity * p.costPrice, 0),
    categoriesCount: categories.length,
    activeSuppliers: suppliers.filter(s => s.isActive).length,
    itemsToReorder: products.filter(p => p.quantity > 0 && p.quantity <= p.minStockLevel).length,
    avgMargin: calculateAverageMargin(products),
    productsTrend: calculateProductsTrend(products, transactions),
    valueTrend: calculateValueTrend(products, transactions),
  }), [products, categories, suppliers, transactions]);
}
```

---

## Test Scenarios

| Test | Type |
|------|------|
| Renders all 8 stat cards with correct values | Component |
| Low Stock card shows correct filtered count | Unit |
| Inventory value calculation is accurate | Unit |
| Average margin handles empty product list (returns 0) | Unit |
| Clicking Low Stock card navigates to filtered products | Component |
| Trend indicators show correct direction and value | Component |
| Cards display in correct grid layout per breakpoint | Component |
