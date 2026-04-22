# StockBase - Inventory Management System

## Project Overview

StockBase is a frontend-only inventory management system for tracking products, categories, suppliers, and stock levels. It allows users to add products through a multi-step wizard, monitor inventory with real-time stats, manage stock with a powerful data table, and receive alerts for low stock items. All data is persisted using the browser's localStorage.

> **For AI Agents**: Before implementing any feature, read `ARCHITECTURE.md` for folder structure and state patterns, `CONVENTIONS.md` for coding standards, and the relevant feature doc in `docs/features/` for detailed specs.

---

## Tech Stack

| Technology | Purpose | Version |
|---|---|---|
| **React** | UI library for building component-based interfaces | 19.x |
| **TypeScript** | Type safety and better developer experience | 5.x (strict mode) |
| **Vite** | Fast build tool and dev server | 6.x |
| **shadcn/ui** | Pre-built accessible UI component library (built on Radix UI) | latest |
| **Tailwind CSS** | Utility-first CSS framework for rapid styling | v4 |
| **Recharts** | Composable chart library for React (bar, line, pie charts) | 2.x |
| **React Hook Form** | Performant form handling with validation | 7.x |
| **Zod** | Schema validation for forms and data | 3.x |
| **date-fns** | Date utility library for date operations | 3.x |
| **Lucide React** | Icon library for clean, consistent icons | latest |
| **react-router-dom** | Client-side routing | v7 |
| **Sonner** | Toast notification library | latest |
| **localStorage** | Browser-based data persistence (no backend) | N/A |

---

## Documentation Map

| Document | What it Covers |
|----------|---------------|
| `PROJECT_OVERVIEW.md` (this file) | Tech stack, data models, routes, storage keys |
| `ARCHITECTURE.md` | Folder structure, state management, data flow, providers, routing |
| `CONVENTIONS.md` | Naming, component patterns, styling, imports, error handling |
| `TESTING_STRATEGY.md` | Test stack, test patterns, factories, coverage targets |
| `features/AddProductWizard.md` | Multi-step product creation form |
| `features/StatsCards.md` | Dashboard stats overview cards |
| `features/ProductTable.md` | Product listing with sort, filter, bulk actions |
| `features/StockToggles.md` | Toggle switches for stock, featured, settings |
| `features/InventoryTabs.md` | Tab navigation between Products/Categories/Suppliers/Alerts |
| `features/ProductDialog.md` | Detail view, edit, restock, and confirmation dialogs |
| `features/CategoryAccordion.md` | Category-wise product grouping with expand/collapse |
| `features/StockAlerts.md` | Alert types, triggers, lifecycle, and management |
| `features/ChartsAndAnalytics.md` | Charts, analytics cards, date range, interactions |

---

## Core Features

| # | Feature | Component Location | Key shadcn/ui Components | Feature Doc |
|---|---|---|---|---|
| 1 | Add Product Wizard | `features/wizard/` | Card, Button, Input, Label, Select, Textarea | [AddProductWizard.md](features/AddProductWizard.md) |
| 2 | Inventory Stats | `features/products/components/StatsCards` | Card, Badge | [StatsCards.md](features/StatsCards.md) |
| 3 | Product Listing | `features/products/components/ProductTable` | Table, Checkbox, DropdownMenu, Badge, Input | [ProductTable.md](features/ProductTable.md) |
| 4 | Stock & Featured Toggles | `features/products/components/` | Switch, Checkbox, Toggle | [StockToggles.md](features/StockToggles.md) |
| 5 | Inventory Tabs | `features/` (cross-feature) | Tabs, Badge | [InventoryTabs.md](features/InventoryTabs.md) |
| 6 | Product Dialogs | `features/products/components/` | Dialog, Sheet, Input, Select | [ProductDialog.md](features/ProductDialog.md) |
| 7 | Category Accordion | `features/categories/components/` | Accordion, Card, Badge | [CategoryAccordion.md](features/CategoryAccordion.md) |
| 8 | Stock Alerts | `features/alerts/components/` | Badge, Alert, Sonner | [StockAlerts.md](features/StockAlerts.md) |
| 9 | Charts & Analytics | `features/analytics/components/` | Recharts (Line, Bar, Pie, Area), Card, Select | [ChartsAndAnalytics.md](features/ChartsAndAnalytics.md) |

---

## Data Models

### Product
```typescript
interface Product {
  id: string;                    // crypto.randomUUID()
  name: string;                  // 3-100 chars
  description: string;           // max 500 chars
  sku: string;                   // unique, auto-generated if blank
  barcode: string;
  categoryId: string;            // FK -> Category.id
  supplierId: string;            // FK -> Supplier.id
  costPrice: number;             // > 0
  sellingPrice: number;          // >= costPrice
  quantity: number;              // >= 0
  minStockLevel: number;         // >= 0, triggers low_stock alert when quantity <= this
  maxStockLevel: number;         // triggers overstock alert when quantity > this
  unit: 'pieces' | 'kg' | 'liters' | 'meters' | 'boxes';
  images: string[];              // URLs or base64 data URIs (see ARCHITECTURE.md image handling)
  isInStock: boolean;            // derived from quantity > 0, or manual override
  isFeatured: boolean;
  isArchived: boolean;           // soft-delete flag, hidden from default views
  tags: string[];
  location: string;              // storage location description
  weight: number | null;
  dimensions: { length: number; width: number; height: number } | null;
  createdAt: string;             // ISO 8601
  updatedAt: string;             // ISO 8601
}
```

### Category
```typescript
interface Category {
  id: string;
  name: string;
  description: string;
  parentId: string | null;       // null = top-level, string = subcategory of parent
  color: string;                 // hex color for UI indicators
  productCount: number;          // denormalized count, update on product add/remove/move
  createdAt: string;
  updatedAt: string;
}
```

### Supplier
```typescript
interface Supplier {
  id: string;
  name: string;
  email: string;
  phone: string;
  company: string;
  address: string;
  leadTimeDays: number;          // used in reorder reminder calculation
  rating: number;                // 1-5 scale
  isActive: boolean;             // inactive suppliers hidden from forms
  productsSupplied: string[];    // array of Product.id references
  createdAt: string;
  updatedAt: string;
}
```

### StockAlert
```typescript
interface StockAlert {
  id: string;
  productId: string;             // FK -> Product.id
  type: 'low_stock' | 'out_of_stock' | 'overstock' | 'expiring_soon';
  severity: 'info' | 'warning' | 'critical';
  message: string;               // human-readable alert text
  isRead: boolean;               // toggled by user
  isDismissed: boolean;          // hides from list, not deleted
  createdAt: string;
}
```

### StockTransaction
```typescript
// Transactions are IMMUTABLE audit logs. They are append-only — never edited or deleted.
interface StockTransaction {
  id: string;
  productId: string;             // FK -> Product.id
  type: 'restock' | 'sale' | 'adjustment' | 'return';
  quantity: number;              // positive = added, for sale/adjustment context matters
  previousQuantity: number;      // snapshot before transaction
  newQuantity: number;           // snapshot after transaction
  note: string;                  // user-provided reason
  createdAt: string;
}
```

### AppSettings
```typescript
interface AppSettings {
  theme: 'light' | 'dark' | 'system';
  lastTab: string;               // last visited tab route (e.g., '/products')
  alertsEnabled: {
    low_stock: boolean;          // default: true
    out_of_stock: boolean;       // default: true
    overstock: boolean;          // default: true
    reorder: boolean;            // default: true
  };
  showToasts: {
    low_stock: boolean;          // default: true
    out_of_stock: boolean;       // default: true
    overstock: boolean;          // default: false
    reorder: boolean;            // default: true
  };
  defaultItemsPerPage: 10 | 25 | 50;  // default: 10
}
```

### Derived Types
```typescript
type StockStatus = 'in_stock' | 'low_stock' | 'out_of_stock';
type AlertSeverity = 'info' | 'warning' | 'critical';
type TransactionType = 'restock' | 'sale' | 'adjustment' | 'return';
type CreateProductInput = Omit<Product, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateProductInput = Partial<CreateProductInput> & { id: string };
type Theme = 'light' | 'dark' | 'system';
```

---

## Business Rules & Calculations

### Stock Status Derivation
```typescript
function getStockStatus(product: Product): StockStatus {
  if (product.quantity === 0) return 'out_of_stock';
  if (product.quantity <= product.minStockLevel) return 'low_stock';
  return 'in_stock';
}
```

### Profit Margin Calculation
```typescript
function calculateProfitMargin(costPrice: number, sellingPrice: number): number {
  if (sellingPrice === 0) return 0;
  return ((sellingPrice - costPrice) / sellingPrice) * 100;
}
```

### Total Inventory Value
```typescript
function calculateInventoryValue(products: Product[]): number {
  return products.reduce((sum, p) => sum + (p.quantity * p.costPrice), 0);
}
```

### Reorder Reminder Logic
```typescript
function shouldTriggerReorderReminder(product: Product, supplier: Supplier, dailySalesRate: number): boolean {
  const daysOfStockLeft = dailySalesRate > 0 ? product.quantity / dailySalesRate : Infinity;
  return daysOfStockLeft <= supplier.leadTimeDays && product.quantity > 0;
}
```

### Average Profit Margin (Stats Card)
```typescript
function calculateAverageMargin(products: Product[]): number {
  if (products.length === 0) return 0;
  const totalMargin = products.reduce((sum, p) => sum + calculateProfitMargin(p.costPrice, p.sellingPrice), 0);
  return totalMargin / products.length;
}
```

### Daily Sales Rate (for forecasting and reorder reminders)
```typescript
function calculateDailySalesRate(productId: string, transactions: StockTransaction[], days: number = 30): number {
  const recentSales = transactions.filter(
    t => t.productId === productId
      && t.type === 'sale'
      && differenceInDays(new Date(), parseISO(t.createdAt)) <= days
  );
  const totalSold = recentSales.reduce((sum, t) => sum + t.quantity, 0);
  return totalSold / days;
}
```

### Stock Depletion Estimate (for low stock forecast chart)
```typescript
function estimateDaysLeft(product: Product, dailySalesRate: number): number | null {
  if (dailySalesRate <= 0) return null;  // no sales history, cannot estimate
  return Math.ceil(product.quantity / dailySalesRate);
}
```

---

## Page Structure & Routing

```
/                        -> Dashboard (stats overview + low stock alerts)
/products                -> Product listing (data table, default)
/products/new            -> Add product wizard (multi-step form)
/products/:id            -> Product detail view
/categories              -> Category management with accordion grouping
/suppliers               -> Supplier listing and management
/alerts                  -> Stock alerts and notifications
/analytics               -> Charts and analytics dashboard
/settings                -> App settings (theme, alert preferences)
```

---

## localStorage Keys

| Key | Data Type | Default |
|---|---|---|
| `stockbase_products` | `Product[]` | `[]` |
| `stockbase_categories` | `Category[]` | Default 5 categories (see below) |
| `stockbase_suppliers` | `Supplier[]` | Default 3 suppliers (see below) |
| `stockbase_alerts` | `StockAlert[]` | `[]` |
| `stockbase_transactions` | `StockTransaction[]` | `[]` (append-only, cap at 10,000 entries) |
| `stockbase_settings` | `AppSettings` | `{ theme: 'system', lastTab: '/products', alertsEnabled: { low_stock: true, out_of_stock: true, overstock: true, reorder: true }, showToasts: { low_stock: true, out_of_stock: true, overstock: false, reorder: true }, defaultItemsPerPage: 10 }` |

---

## Default Categories (Seed Data)

Every new setup starts with these categories:

| Name | Description | Color |
|------|-------------|-------|
| Electronics | Gadgets, devices, components | `#3b82f6` (blue) |
| Clothing | Apparel, accessories, footwear | `#8b5cf6` (purple) |
| Food & Beverages | Perishables, packaged goods, drinks | `#22c55e` (green) |
| Office Supplies | Stationery, furniture, equipment | `#f59e0b` (amber) |
| Raw Materials | Unprocessed goods, ingredients | `#6b7280` (gray) |

---

## Default Suppliers (Seed Data)

Every new setup starts with these suppliers:

| Name | Company | Email | Lead Time | Rating |
|------|---------|-------|-----------|--------|
| John Smith | TechSupply Co | john@techsupply.co | 5 days | 4 |
| Sarah Lee | FreshGoods Inc | sarah@freshgoods.com | 3 days | 5 |
| Mike Chen | GlobalParts Ltd | mike@globalparts.com | 7 days | 3 |

---

## Entity Lifecycle Summary

| Entity | Create | Update | Delete | Notes |
|--------|--------|--------|--------|-------|
| Product | Wizard form | Edit dialog, inline edit, restock, toggles | Soft-delete (archive) or hard-delete | Archive sets `isArchived: true`; hard-delete removes from array |
| Category | Dialog form | Dialog form | Hard-delete with product reassignment | Cannot delete if has subcategories; must reassign products first |
| Supplier | Dialog form | Dialog form | Hard-delete with confirmation | Cannot delete if products reference this supplier |
| StockAlert | Auto-created by alert engine | Mark read, dismiss | Auto-cleared when condition resolves; hard-deleted when product deleted | Dismissed = soft-hidden; auto-cleared = removed from state |
| StockTransaction | Auto-created on any stock change | Never | Never | Immutable append-only audit log |
