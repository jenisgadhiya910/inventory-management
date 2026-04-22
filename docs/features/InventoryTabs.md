# Inventory Tabs (Products, Categories, Suppliers, Alerts)

## What It Does

Navigation tabs let users switch between the main sections of the inventory system. Each section is a separate route (not a tab component) for deep linking and browser history support.

> **Implementation note:** Navigation is handled by `react-router-dom` `NavLink` components in `AppLayout`, NOT by shadcn `Tabs` component. Each "tab" is a route. See [ARCHITECTURE.md](../ARCHITECTURE.md) for AppLayout and routing details.

---

## Component Architecture

```
# Tabs are implemented via react-router-dom routes, not shadcn Tabs component.
# Each tab is a separate route, with shared layout.

app/routes.tsx                    # Route definitions
components/shared/
├── AppLayout.tsx                 # Sidebar nav + main content area
├── TabNavigation.tsx             # Tab bar component with badges
└── TabLink.tsx                   # Single tab link with icon, label, badge

features/products/                # /products route
features/categories/              # /categories route
features/suppliers/               # /suppliers route
features/alerts/                  # /alerts route
```

### Why Routes Instead of Tabs Component

- Deep linking: users can bookmark `/alerts` directly
- Browser back/forward navigation works
- Each tab's state is isolated in its own route component
- URL reflects current view for shareability

---

## Component Hierarchy

```
<AppLayout>
  <Sidebar>
    <TabNavigation>
      <TabLink to="/products" icon={Package} label="Products" badge={productCount} />
      <TabLink to="/categories" icon={FolderTree} label="Categories" badge={categoryCount} />
      <TabLink to="/suppliers" icon={Truck} label="Suppliers" badge={activeSupplierCount} />
      <TabLink to="/alerts" icon={Bell} label="Alerts" badge={unreadAlertCount} badgeVariant="critical" />
    </TabNavigation>
  </Sidebar>
  <main>
    <Outlet />   {/* Renders the matched route component */}
  </main>
</AppLayout>
```

---

## Tab Definitions

### TabLink Props

```typescript
interface TabLinkProps {
  to: string;
  icon: LucideIcon;
  label: string;
  badge?: number;
  badgeVariant?: 'default' | 'critical';  // critical = red background
}
```

### The Four Tabs

| Tab | Route | Icon | Badge Value | Badge Variant |
|-----|-------|------|-------------|---------------|
| Products | `/products` | `Package` | `products.length` | default |
| Categories | `/categories` | `FolderTree` | `categories.length` | default |
| Suppliers | `/suppliers` | `Truck` | `suppliers.filter(s => s.isActive).length` | default |
| Alerts | `/alerts` | `Bell` | `alerts.filter(a => !a.isRead && !a.isDismissed).length` | critical (if any unread) |

---

## Products Tab (`/products`)

- Renders `<ProductListing>` component
- Contains: `ProductTableToolbar`, `ProductTable`, `ProductPagination`
- "Add Product" button in top-right -> navigates to `/products/new`
- See [ProductTable.md](ProductTable.md) for full details
- Filter/search state managed by `useProductFilters` hook (local to this route)

---

## Categories Tab (`/categories`)

- Renders `<CategoryManagement>` component
- Contains: search bar, sort dropdown, `CategoryAccordion`
- "Add Category" button in top-right -> opens category creation dialog
- See [CategoryAccordion.md](CategoryAccordion.md) for full details

### Category Management Features

- Add/edit/delete categories via dialogs
- Subcategory support (parent/child)
- Drag to reorder (or sort dropdown)
- Deleting requires product reassignment

---

## Suppliers Tab (`/suppliers`)

- Renders `<SupplierManagement>` component

### Supplier Table Columns

| Column | Field | Sortable | Notes |
|--------|-------|----------|-------|
| Supplier Name | `name` | Yes | Click -> supplier detail |
| Company | `company` | Yes | |
| Email | `email` | No | `mailto:` link |
| Phone | `phone` | No | `tel:` link |
| Lead Time | `leadTimeDays` | Yes | "{n} days" format |
| Rating | `rating` | Yes | Star display (1-5) |
| Products | `productsSupplied.length` | Yes | Count badge |
| Status | `isActive` | Yes | Toggle switch |

### Supplier Actions

> **Note:** Supplier detail is a **Dialog overlay** (not a separate `/suppliers/:id` route). Clicking a supplier name opens a detail dialog. All supplier management happens via dialogs on the `/suppliers` route.

| Action | Behavior |
|--------|----------|
| View Supplier | Click supplier name -> opens SupplierDetailDialog (read-only) |
| Add Supplier | "Add Supplier" button -> opens SupplierFormDialog |
| Edit | Three-dot menu or detail dialog "Edit" button -> opens SupplierFormDialog pre-filled |
| Deactivate | Toggle `isActive` to false (see [StockToggles.md](StockToggles.md)), hides from product form dropdowns |
| Delete | Confirmation dialog. **Blocked** if products reference this supplier — show count and "Reassign products first". |

### Supplier Form Fields

```typescript
// Zod schema
const supplierSchema = z.object({
  name: z.string().min(2, 'Supplier name is required'),
  email: z.string().email('Invalid email address'),
  phone: z.string().min(5, 'Phone number is required'),
  company: z.string().min(2, 'Company name is required'),
  address: z.string().optional().default(''),
  leadTimeDays: z.number().min(1, 'Lead time must be at least 1 day'),
  rating: z.number().min(1).max(5).default(3),
  isActive: z.boolean().default(true),
});
```

---

## Alerts Tab (`/alerts`)

- Renders `<AlertsPage>` component
- Sorted by severity: critical first, then warning, then info
- See [StockAlerts.md](StockAlerts.md) for full details

### Alert List Features

- Filter by type, severity, read/unread status
- "Mark All as Read" and "Dismiss All" bulk actions
- Individual alert actions: Restock, Mark as Read, Dismiss
- Click product name -> navigate to product detail

---

## Tab Behavior

| Behavior | Implementation |
|----------|---------------|
| Active tab highlighted | `NavLink` with `className` callback checking `isActive` |
| URL reflects active tab | Direct route mapping (`/products`, `/categories`, etc.) |
| Last tab remembered | On navigate, save current path to `stockbase_settings.lastTab`. On app open at `/`, redirect to `lastTab` if set. **URL always takes precedence** — direct URL access (e.g., `/alerts`) overrides `lastTab`. |
| Independent filter state | Each route component manages its own filter state via hooks (resets on unmount) |
| Keyboard navigation | Arrow keys navigate between tab links when tab bar is focused |
| Badge updates in real-time | Badges read from context, re-render on state change |

---

## Responsive Behavior

| Screen Size | Layout |
|---|---|
| Desktop (`lg:`) | Sidebar with icons + labels, main content area |
| Tablet (`md:`) | Sidebar with icons + labels, slightly narrower |
| Mobile (default) | Bottom tab bar with icons only (labels hidden) |

```typescript
// Mobile: bottom tab bar
<nav className="fixed bottom-0 left-0 right-0 flex justify-around border-t bg-background p-2 md:hidden">
  {tabs.map(tab => <TabLink key={tab.to} {...tab} showLabel={false} />)}
</nav>

// Desktop: sidebar
<aside className="hidden md:flex md:w-64 md:flex-col md:border-r">
  {tabs.map(tab => <TabLink key={tab.to} {...tab} showLabel={true} />)}
</aside>
```

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Direct URL access to `/alerts` | Renders alerts tab directly, no redirect needed |
| Tab badge shows 0 | Badge hidden when count is 0 |
| Unknown route (e.g., `/foo`) | Redirect to `/` (dashboard) |
| Rapid tab switching | Route components unmount/mount; filter state resets per design |

---

## Test Scenarios

| Test | Type |
|------|------|
| All 4 tabs render and navigate correctly | E2E |
| Tab badges show correct counts | Component |
| Active tab is highlighted based on URL | Component |
| Mobile renders bottom tab bar, desktop renders sidebar | Component |
| Alert badge turns red when critical alerts exist | Component |
| Keyboard arrow navigation between tabs | Component |
| Direct URL access renders correct tab content | E2E |
