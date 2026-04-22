# Product Table (Product Listing)

## What It Does

The product table is the main view for browsing and managing all products in inventory. It presents products in a spreadsheet-like format with sorting, filtering, searching, bulk actions, inline editing, and pagination.

> **Dependencies:** Reads from `ProductContext`, `CategoryContext` (resolve category names), `SupplierContext`. Writes to `ProductContext` (inline edits, bulk actions), `AlertContext` (via `useAlertEngine` on quantity changes), `TransactionContext` (on inline quantity edits). Opens [ProductDialog](ProductDialog.md) for edit/restock/delete. Links to [AddProductWizard](AddProductWizard.md) via "Add Product" button. Status badge click uses [StockToggles](StockToggles.md) toggle behavior.

> **URL:** `/products` — Supports query params:
> - `?status=low_stock` — Pre-applies Low Stock filter (linked from [StatsCards](StatsCards.md))
> - `?status=out_of_stock` — Pre-applies Out of Stock filter (linked from [StatsCards](StatsCards.md))

---

## Component Architecture

```
features/products/components/
├── ProductTable.tsx              # Main table component with data orchestration
├── ProductTableToolbar.tsx       # Search bar, filter dropdowns, filter chips
├── ProductTableHeader.tsx        # Column headers with sort controls
├── ProductRow.tsx                # Single product row (memoized with React.memo)
├── ProductBulkActions.tsx        # Toolbar shown when products are selected
├── ProductPagination.tsx         # Page controls and items-per-page selector
├── InlineQuantityEdit.tsx        # Click-to-edit +/- control for quantity
├── InlinePriceEdit.tsx           # Click-to-edit for selling price
├── ProductEmptyState.tsx         # Empty state messaging
└── ProductCardLayout.tsx         # Mobile card-based alternative layout

features/products/hooks/
├── useProductFilters.ts          # Filter state management (search, category, status, price range)
├── useProductSort.ts             # Sort column + direction state
├── useProductPagination.ts       # Page number, items per page, derived slice
└── useProductSelection.ts        # Checkbox selection state, select all logic
```

### Component Hierarchy

```
<ProductListing>                   # Route component at /products
  <ProductTableToolbar
    filters={filters}
    onFilterChange={setFilters}
    onSearch={setSearchTerm}
  />
  <ProductBulkActions              # Conditional: visible when selectedIds.length > 0
    selectedIds={selectedIds}
    onBulkAction={handleBulkAction}
  />
  <ProductTable>
    <ProductTableHeader
      columns={columns}
      sortColumn={sort.column}
      sortDirection={sort.direction}
      onSort={handleSort}
      allSelected={allSelected}
      onSelectAll={handleSelectAll}
    />
    {paginatedProducts.map(product => (
      <ProductRow
        key={product.id}
        product={product}
        selected={selectedIds.includes(product.id)}
        onSelect={handleSelect}
        onInlineEdit={handleInlineEdit}
      />
    ))}
  </ProductTable>
  <ProductPagination
    total={filteredProducts.length}
    page={page}
    perPage={perPage}
    onPageChange={setPage}
    onPerPageChange={setPerPage}
  />
  {filteredProducts.length === 0 && <ProductEmptyState type={emptyStateType} />}
</ProductListing>
```

---

## Table Columns

| Column | Field | Sortable | Width | Notes |
|--------|-------|----------|-------|-------|
| Checkbox | - | No | 40px | Select for bulk actions |
| Image | `images[0]` | No | 48px | Thumbnail, fallback placeholder |
| Product Name | `name` | Yes | flex | Click -> `/products/:id` |
| SKU | `sku` | Yes | 120px | Monospace font |
| Category | `categoryId` | Yes | 140px | Resolved to category name |
| Stock Quantity | `quantity` | Yes | 120px | Inline editable +/- control |
| Cost Price | `costPrice` | Yes | 100px | Formatted as currency |
| Selling Price | `sellingPrice` | Yes | 100px | Inline editable, formatted as currency |
| Status | derived | Yes | 100px | Badge: In Stock / Low Stock / Out of Stock |
| Actions | - | No | 48px | Three-dot `DropdownMenu` |

---

## Sorting

```typescript
interface SortState {
  column: keyof Product | null;
  direction: 'asc' | 'desc' | null;
}
```

- Click column header: sets sort to `asc`
- Click again: toggles to `desc`
- Click third time: clears sort (returns to default: newest first by `createdAt`)
- Visual: `ArrowUp` / `ArrowDown` icon next to active sort column

```typescript
function sortProducts(products: Product[], sort: SortState): Product[] {
  if (!sort.column) return products.sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime());
  return [...products].sort((a, b) => {
    const aVal = a[sort.column!];
    const bVal = b[sort.column!];
    const comparison = typeof aVal === 'string' ? aVal.localeCompare(bVal as string) : (aVal as number) - (bVal as number);
    return sort.direction === 'desc' ? -comparison : comparison;
  });
}
```

---

## Filtering

### Filter State

```typescript
interface ProductFilters {
  searchTerm: string;              // debounced, 300ms
  categoryIds: string[];           // multi-select
  supplierIds: string[];           // multi-select
  status: StockStatus | null;      // single select or null for all
  priceRange: { min: number | null; max: number | null };
  showArchived: boolean;
}

// Initialize from URL query params
const [searchParams] = useSearchParams();
const initialStatus = searchParams.get('status') as StockStatus | null;

const [filters, setFilters] = useState<ProductFilters>({
  searchTerm: '',
  categoryIds: [],
  supplierIds: [],
  status: initialStatus,          // pre-filled from ?status= query param
  priceRange: { min: null, max: null },
  showArchived: false,
});
```

### Filter Application Order

```typescript
function applyFilters(products: Product[], filters: ProductFilters): Product[] {
  return products
    .filter(p => !filters.searchTerm || matchesSearch(p, filters.searchTerm))
    .filter(p => filters.categoryIds.length === 0 || filters.categoryIds.includes(p.categoryId))
    .filter(p => filters.supplierIds.length === 0 || filters.supplierIds.includes(p.supplierId))
    .filter(p => !filters.status || getStockStatus(p) === filters.status)
    .filter(p => filters.priceRange.min === null || p.sellingPrice >= filters.priceRange.min)
    .filter(p => filters.priceRange.max === null || p.sellingPrice <= filters.priceRange.max);
}

function matchesSearch(product: Product, term: string): boolean {
  const lower = term.toLowerCase();
  return product.name.toLowerCase().includes(lower)
    || product.sku.toLowerCase().includes(lower)
    || product.description.toLowerCase().includes(lower);
}
```

### Active Filter Chips

Active filters render as removable chips below the toolbar:
```typescript
<div className="flex flex-wrap gap-2">
  {filters.categoryIds.map(id => (
    <Badge key={id} variant="secondary" className="cursor-pointer" onClick={() => removeFilter('category', id)}>
      {getCategoryName(id)} <X className="ml-1 h-3 w-3" />
    </Badge>
  ))}
  {hasActiveFilters && <Button variant="ghost" size="sm" onClick={clearAllFilters}>Clear all</Button>}
</div>
```

---

## Row Actions (Three-Dot Menu)

| Action | Behavior |
|--------|----------|
| Edit | Opens Edit Product dialog |
| Restock | Opens Restock Confirmation dialog |
| Duplicate | Creates copy with new ID/SKU, name appended " (Copy)" |
| Archive | Soft-deletes product (sets `isArchived: true`) |
| Delete | Opens Confirmation dialog, then removes from state |

---

## Bulk Actions

Visible when `selectedIds.length > 0`:

| Action | Behavior |
|--------|----------|
| Change Category | Opens category `Select`, moves all selected products |
| Update Status | Toggle in-stock/out-of-stock for all selected |
| Export | Generates CSV download of selected products |
| Delete | Confirmation dialog -> removes all selected from state |

### Select All Logic

```typescript
// Header checkbox behavior
const allSelected = paginatedProducts.length > 0 && paginatedProducts.every(p => selectedIds.includes(p.id));
const someSelected = paginatedProducts.some(p => selectedIds.includes(p.id)) && !allSelected;
// someSelected -> indeterminate checkbox state
```

---

## Pagination

```typescript
interface PaginationState {
  page: number;           // 1-indexed
  perPage: number;        // 10 | 25 | 50 | Infinity ('All')
}
```

- Default: 10 per page
- "Showing 1-10 of 156 products" label
- Previous/Next buttons + page number buttons
- Changing filters resets to page 1

---

## Inline Editing

### Quantity (+/- Control)

```typescript
// InlineQuantityEdit.tsx
// Click quantity cell -> shows [-] [value] [+] controls
// Each click dispatches UPDATE_PRODUCT with new quantity
// Also creates a StockTransaction { type: 'adjustment' }
// Also triggers alert evaluation
```

### Selling Price

```typescript
// InlinePriceEdit.tsx
// Click price cell -> shows Input with current value
// Blur or Enter -> save new price
// Escape -> cancel edit
// Validate: must be >= costPrice
```

Changes save immediately via `useProducts().updateProduct()`.

---

## Stock Status Badges

| Status | Condition | Badge |
|--------|-----------|-------|
| In Stock | `quantity > minStockLevel` | `<Badge variant="default" className="bg-green-100 text-green-800">In Stock</Badge>` |
| Low Stock | `quantity > 0 && quantity <= minStockLevel` | `<Badge variant="default" className="bg-amber-100 text-amber-800">Low Stock</Badge>` |
| Out of Stock | `quantity === 0` | `<Badge variant="destructive">Out of Stock</Badge>` |

Progress bar next to quantity: `(quantity / maxStockLevel) * 100`%, capped at 100%.

---

## Empty States

| Situation | Message | Action |
|-----------|---------|--------|
| No products at all | "No products yet. Add your first product to get started." | `<Button>` -> `/products/new` |
| Filters yield no results | "No products match your filters." | `<Button variant="link">Clear filters</Button>` |
| All products archived | "All products are archived. Toggle 'Show archived' to view them." | Toggle button |

---

## Responsive Behavior

| Screen Size | Layout |
|---|---|
| Desktop (`lg:`) | Full table with all columns |
| Tablet (`md:`) | Hide Image + SKU columns, enable horizontal scroll |
| Mobile (default) | Switch to `ProductCardLayout` — stacked cards with key info |

```typescript
const isMobile = useMediaQuery('(max-width: 768px)');
return isMobile ? <ProductCardLayout products={products} /> : <ProductTable products={products} />;
```

---

## Performance

| Concern | Strategy |
|---------|----------|
| 1000+ products | Use `react-window` `FixedSizeList` for virtualized rows |
| Re-renders on filter change | `React.memo` on `ProductRow`, only re-render if product data or selection changes |
| Search debounce | `useDebounce(searchTerm, 300)` |
| Sort/filter chaining | `useMemo` with `[products, filters, sort]` dependencies |
| Bulk select performance | `Set<string>` for `selectedIds` instead of array `.includes()` |

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Product with no image | Show placeholder icon (Package icon from Lucide) |
| Product with deleted category | Show "Uncategorized" in category column |
| Product with inactive supplier | Show supplier name in muted text |
| Inline edit to negative quantity | Clamp to 0, trigger out_of_stock alert |
| Inline price below cost price | Show inline error, revert on blur |
| Select all + change page | Selection persists across pages |
| Delete last product on page | Navigate to previous page |

## Accessibility

- Sort headers: `<th aria-sort="ascending">` / `"descending"` / `"none"`. Click header to cycle.
- Header checkbox: `aria-checked="mixed"` when some (not all) products are selected.
- Product name links: `<a>` or `<button>` with clear text, not wrapping entire row.
- Inline edit mode: `aria-label="Edit quantity for {productName}"` on the +/- controls.
- Status badges: include text label inside badge (never color-only).
- Pagination: `<nav aria-label="Product table pagination">` wrapping page buttons.
- Empty state: `role="status"` with `aria-live="polite"` so screen readers announce filter results.
- Filter chips: each chip is a `<button>` with `aria-label="Remove filter: {filterName}"`.

---

| Test | Type |
|------|------|
| Renders product list with correct columns | Component |
| Sorting toggles through asc/desc/none | Component |
| Search filters products by name, SKU, description | Component |
| Category filter shows correct products | Component |
| Pagination shows correct subset | Component |
| Bulk selection and "Select All" work correctly | Component |
| Inline quantity edit updates product and creates transaction | Integration |
| Empty state renders when no products | Component |
| Mobile layout switches to card view | Component |
| Full filter + sort + paginate flow | E2E |
