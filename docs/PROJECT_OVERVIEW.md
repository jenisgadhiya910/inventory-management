# StockBase - Inventory Management System

## Project Overview

StockBase is a frontend-only inventory management system for tracking products, categories, suppliers, and stock levels. It allows users to add products through a multi-step wizard, monitor inventory with real-time stats, manage stock with a powerful data table, and receive alerts for low stock items. All data is persisted using the browser's localStorage.

---

## Tech Stack

| Technology | Purpose |
|---|---|
| **React 19** | UI library for building component-based interfaces |
| **TypeScript** | Type safety and better developer experience |
| **Vite** | Fast build tool and dev server |
| **shadcn/ui** | Pre-built accessible UI component library (built on Radix UI) |
| **Tailwind CSS v4** | Utility-first CSS framework for rapid styling |
| **Recharts** | Composable chart library for React (bar, line, pie charts) |
| **date-fns** | Date utility library for date operations |
| **Lucide React** | Icon library for clean, consistent icons |
| **localStorage** | Browser-based data persistence (no backend) |

---

## Core Features

| # | Feature | Component Type | Key shadcn/ui Components |
|---|---|---|---|
| 1 | Add Product Wizard | Multi-step form | Card, Button, Input, Label, Select, Textarea |
| 2 | Inventory Stats | Stats cards | Card, Badge |
| 3 | Product Listing | Data table | Table, Checkbox, DropdownMenu, Badge, Input |
| 4 | Stock & Featured Toggles | Toggles | Switch, Checkbox, Toggle |
| 5 | Products / Categories / Suppliers / Alerts | Tabs | Tabs, Badge |
| 6 | Edit Product & Restock Confirm | Modal / Dialog | Dialog, Sheet, Input, Select |
| 7 | Category-wise Grouping | Accordion | Accordion, Card, Badge |
| 8 | Low Stock Warnings | Badge / Alerts | Badge, Alert, Sonner |
| 9 | Charts & Analytics | Data visualization | Recharts (Line, Bar, Pie, Area), Card, Select |

---

## Data Models

### Product
```typescript
interface Product {
  id: string;
  name: string;
  description: string;
  sku: string;
  barcode: string;
  categoryId: string;
  supplierId: string;
  costPrice: number;
  sellingPrice: number;
  quantity: number;
  minStockLevel: number;
  maxStockLevel: number;
  unit: 'pieces' | 'kg' | 'liters' | 'meters' | 'boxes';
  images: string[];
  isInStock: boolean;
  isFeatured: boolean;
  tags: string[];
  location: string;
  weight: number | null;
  dimensions: { length: number; width: number; height: number } | null;
  createdAt: string;
  updatedAt: string;
}
```

### Category
```typescript
interface Category {
  id: string;
  name: string;
  description: string;
  parentId: string | null;
  color: string;
  productCount: number;
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
  leadTimeDays: number;
  rating: number;
  isActive: boolean;
  productsSupplied: string[];
  createdAt: string;
  updatedAt: string;
}
```

### StockAlert
```typescript
interface StockAlert {
  id: string;
  productId: string;
  type: 'low_stock' | 'out_of_stock' | 'overstock' | 'expiring_soon';
  severity: 'info' | 'warning' | 'critical';
  message: string;
  isRead: boolean;
  isDismissed: boolean;
  createdAt: string;
}
```

### StockTransaction
```typescript
interface StockTransaction {
  id: string;
  productId: string;
  type: 'restock' | 'sale' | 'adjustment' | 'return';
  quantity: number;
  previousQuantity: number;
  newQuantity: number;
  note: string;
  createdAt: string;
}
```

---

## Page Structure

```
/                        → Dashboard (stats overview + low stock alerts)
/products/new            → Add product wizard (multi-step form)
/products                → Product listing (data table, default)
/products/:id            → Product detail view
/categories              → Category management with accordion grouping
/suppliers               → Supplier listing and management
/alerts                  → Stock alerts and notifications
/analytics               → Charts and analytics dashboard
```

---

## localStorage Keys

| Key | Data |
|---|---|
| `stockbase_products` | Array of all products |
| `stockbase_categories` | Array of all categories |
| `stockbase_suppliers` | Array of all suppliers |
| `stockbase_alerts` | Array of stock alerts |
| `stockbase_transactions` | Array of stock transactions |
| `stockbase_settings` | User preferences (theme, default view, alert thresholds) |

---

## Default Categories

Every new setup starts with these categories:
1. **Electronics** - Gadgets, devices, components
2. **Clothing** - Apparel, accessories, footwear
3. **Food & Beverages** - Perishables, packaged goods, drinks
4. **Office Supplies** - Stationery, furniture, equipment
5. **Raw Materials** - Unprocessed goods, ingredients