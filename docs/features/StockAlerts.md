# Stock Alerts (Low Stock Warnings & Notifications)

## What It Does

Stock alerts notify users when products need attention — running low, out of stock, overstocked, or needing reorder. Alerts appear as badges, banners, toasts, and in a dedicated alerts list so critical inventory issues are never missed.

> **Dependencies:** The alert engine (`useAlertEngine` in `features/alerts/hooks/`) reads from `ProductContext` and `SettingsContext` to evaluate conditions. It writes to `AlertContext`. It is called by any code that mutates product quantities: [AddProductWizard](AddProductWizard.md) (on submit), [ProductTable](ProductTable.md) (inline edits), [ProductDialog](ProductDialog.md) (restock/edit), and [StockToggles](StockToggles.md) (stock status toggle).

> **Restock button on alert cards:** Opens the [RestockDialog](ProductDialog.md#3-restock-confirmation-dialog-medium) as a Dialog overlay on the `/alerts` page (no navigation). The dialog is the same `RestockDialog` component used elsewhere.

---

## Component Architecture

```
features/alerts/
├── components/
│   ├── AlertsPage.tsx            # Full alert list page (/alerts route)
│   ├── AlertList.tsx             # Filtered, sorted list of alert cards
│   ├── AlertCard.tsx             # Single alert with icon, message, actions
│   ├── AlertBanner.tsx           # Dashboard critical alert banner
│   ├── AlertBadge.tsx            # Small count badge for nav/tabs
│   └── AlertFilters.tsx          # Filter bar: type, severity, status
├── hooks/
│   ├── useAlerts.ts              # CRUD operations via AlertContext
│   └── useAlertEngine.ts         # Alert creation/clearing logic
└── context/
    └── AlertContext.tsx           # State + reducer for alerts
```

---

## Alert Types

### Type Definitions

```typescript
interface AlertTypeConfig {
  type: StockAlert['type'];
  severity: AlertSeverity;
  icon: LucideIcon;
  color: string;             // Tailwind color class
  trigger: (product: Product, supplier?: Supplier) => boolean;
  message: (product: Product) => string;
}

const ALERT_CONFIGS: AlertTypeConfig[] = [
  {
    type: 'low_stock',
    severity: 'warning',
    icon: AlertTriangle,
    color: 'text-amber-500',
    trigger: (p) => p.quantity > 0 && p.quantity <= p.minStockLevel,
    message: (p) => `${p.name} has only ${p.quantity} ${p.unit} left (minimum: ${p.minStockLevel})`,
  },
  {
    type: 'out_of_stock',
    severity: 'critical',
    icon: XCircle,
    color: 'text-red-500',
    trigger: (p) => p.quantity === 0,
    message: (p) => `${p.name} is out of stock`,
  },
  {
    type: 'overstock',
    severity: 'info',
    icon: ArrowUpCircle,
    color: 'text-blue-500',
    trigger: (p) => p.maxStockLevel > 0 && p.quantity > p.maxStockLevel,
    message: (p) => `${p.name} has ${p.quantity} ${p.unit} (maximum: ${p.maxStockLevel})`,
  },
  {
    type: 'expiring_soon',  // reorder reminder
    severity: 'warning',
    icon: Clock,
    color: 'text-amber-500',
    trigger: (p, supplier) => {
      if (!supplier) return false;
      const dailySalesRate = calculateDailySalesRate(p.id);
      const daysLeft = dailySalesRate > 0 ? p.quantity / dailySalesRate : Infinity;
      return daysLeft <= supplier.leadTimeDays && p.quantity > 0;
    },
    message: (p) => `Order ${p.name} now — current stock will last approximately ${estimateDaysLeft(p)} days`,
  },
];
```

---

## Alert Engine

The alert engine evaluates a product and creates/clears alerts as needed. It runs:
- After any product quantity change (restock, sale, inline edit, wizard submit)
- After product creation
- After product deletion (clears all alerts for that product)

```typescript
// hooks/useAlertEngine.ts
function useAlertEngine() {
  const { alerts, addAlert, removeAlert } = useAlerts();
  const { settings } = useSettings();

  const evaluateProduct = (product: Product, supplier?: Supplier) => {
    ALERT_CONFIGS.forEach(config => {
      // Skip if alert type is disabled in settings
      if (!settings.alertsEnabled[config.type]) return;

      const existingAlert = alerts.find(
        a => a.productId === product.id && a.type === config.type && !a.isDismissed
      );

      const shouldAlert = config.trigger(product, supplier);

      if (shouldAlert && !existingAlert) {
        // Create new alert
        addAlert({
          id: generateId(),
          productId: product.id,
          type: config.type,
          severity: config.severity,
          message: config.message(product),
          isRead: false,
          isDismissed: false,
          createdAt: new Date().toISOString(),
        });
        // Show toast for new alerts
        toast[config.severity === 'critical' ? 'error' : 'warning'](config.message(product), {
          action: { label: 'View', onClick: () => navigate(`/alerts`) },
        });
      } else if (!shouldAlert && existingAlert) {
        // Condition resolved — clear the alert
        removeAlert(existingAlert.id);
      }
    });
  };

  const clearAlertsForProduct = (productId: string) => {
    alerts
      .filter(a => a.productId === productId)
      .forEach(a => removeAlert(a.id));
  };

  return { evaluateProduct, clearAlertsForProduct };
}
```

### Deduplication

One alert per (productId, type) combination. Before creating, check:
```typescript
const exists = alerts.some(a => a.productId === product.id && a.type === config.type && !a.isDismissed);
```

---

## Where Alerts Appear

### 1. Dashboard Banner

```typescript
// AlertBanner.tsx — shown at top of Dashboard when critical alerts exist
const criticalAlerts = alerts.filter(a => a.severity === 'critical' && !a.isDismissed);

{criticalAlerts.length > 0 && (
  <Alert variant="destructive" className="mb-4">
    <AlertTriangle className="h-4 w-4" />
    <AlertTitle>{criticalAlerts.length} critical alert{criticalAlerts.length !== 1 ? 's' : ''}</AlertTitle>
    <AlertDescription>
      You have products that are out of stock.
      <Button variant="link" onClick={() => navigate('/alerts')}>View All</Button>
    </AlertDescription>
  </Alert>
)}
```

### 2. Navigation Badge

```typescript
// AlertBadge.tsx
const unreadCount = alerts.filter(a => !a.isRead && !a.isDismissed).length;
const hasCritical = alerts.some(a => a.severity === 'critical' && !a.isRead && !a.isDismissed);

{unreadCount > 0 && (
  <Badge variant={hasCritical ? 'destructive' : 'secondary'} className="ml-2">
    {unreadCount}
  </Badge>
)}
```

Shown on: Alerts tab link, bell icon in app header.

### 3. Product Table

Products with active alerts show a warning icon next to their name:
```typescript
{activeAlerts.some(a => a.productId === product.id) && (
  <AlertTriangle className="h-4 w-4 text-amber-500 ml-1" />
)}
```

### 4. Toast Notifications

Triggered by the alert engine when a new alert is created. See `evaluateProduct` above.

### 5. Alerts Page (`/alerts`)

Full dedicated page — see Alert List Layout below.

---

## Alert List Layout (Alerts Page)

### AlertCard Props

```typescript
interface AlertCardProps {
  alert: StockAlert;
  productName: string;
  onRestock: (productId: string) => void;
  onMarkRead: (alertId: string) => void;
  onDismiss: (alertId: string) => void;
}
```

### Card Content

```
<Card className={cn('border-l-4', severityBorderColor[alert.severity], !alert.isRead && 'bg-muted/50')}>
  <div className="flex items-start gap-3 p-4">
    <SeverityIcon type={alert.type} />
    <div className="flex-1">
      <button onClick={() => navigate(`/products/${alert.productId}`)}>
        {productName}
      </button>
      <p className="text-sm text-muted-foreground">{alert.message}</p>
      <time className="text-xs text-muted-foreground">{formatDistanceToNow(alert.createdAt)} ago</time>
    </div>
    <div className="flex gap-2">
      <Button size="sm" onClick={() => onRestock(alert.productId)}>Restock</Button>
      <Button size="sm" variant="ghost" onClick={() => onMarkRead(alert.id)}>
        {alert.isRead ? 'Unread' : 'Mark Read'}
      </Button>
      <Button size="sm" variant="ghost" onClick={() => onDismiss(alert.id)}>Dismiss</Button>
    </div>
  </div>
</Card>
```

---

## Alert Filtering

```typescript
interface AlertFilters {
  type: StockAlert['type'] | null;         // null = all types
  severity: AlertSeverity | null;           // null = all severities
  status: 'unread' | 'read' | 'dismissed' | null;  // null = active (not dismissed)
}
```

Default filter: show active (not dismissed) alerts, sorted by severity then date.

```typescript
function filterAlerts(alerts: StockAlert[], filters: AlertFilters): StockAlert[] {
  return alerts
    .filter(a => !filters.type || a.type === filters.type)
    .filter(a => !filters.severity || a.severity === filters.severity)
    .filter(a => {
      if (filters.status === 'unread') return !a.isRead && !a.isDismissed;
      if (filters.status === 'read') return a.isRead && !a.isDismissed;
      if (filters.status === 'dismissed') return a.isDismissed;
      return !a.isDismissed; // default: active
    })
    .sort((a, b) => {
      const severityOrder = { critical: 0, warning: 1, info: 2 };
      const severityDiff = severityOrder[a.severity] - severityOrder[b.severity];
      if (severityDiff !== 0) return severityDiff;
      return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime();
    });
}
```

---

## Bulk Actions

| Action | Behavior |
|--------|----------|
| Mark All as Read | `alerts.filter(a => !a.isRead).forEach(a => markAsRead(a.id))` |
| Dismiss All | Confirmation dialog -> `alerts.filter(a => !a.isDismissed).forEach(a => dismiss(a.id))` |

---

## Alert Lifecycle

```
1. CREATED      — Alert engine detects condition, creates alert (isRead: false, isDismissed: false)
2. TOAST        — Toast notification shown to user
3. UNREAD       — Alert visible in list with bold/highlighted styling
4. READ         — User clicks "Mark Read" or views the alert (isRead: true)
5. DISMISSED    — User clicks "Dismiss" (isDismissed: true, hidden from default view)
6. AUTO-CLEARED — Condition resolved (e.g., product restocked), alert removed entirely
```

**Key distinction:** Dismissed alerts are soft-deleted (still in storage, viewable via filter). Auto-cleared alerts are hard-deleted (removed from state and localStorage).

---

## Alert Counts

| Location | Calculation |
|----------|-------------|
| App header bell icon | `alerts.filter(a => !a.isRead && !a.isDismissed).length` |
| Alerts tab badge | Same as above; red variant if any `severity === 'critical'` |
| Dashboard banner | `alerts.filter(a => a.severity === 'critical' && !a.isDismissed).length` |
| Stats card (Low Stock) | `products.filter(p => p.quantity > 0 && p.quantity <= p.minStockLevel).length` |

---

## Alert Settings

```typescript
// In stockbase_settings
interface AlertSettings {
  alertsEnabled: {
    low_stock: boolean;     // default: true
    out_of_stock: boolean;  // default: true
    overstock: boolean;     // default: true
    reorder: boolean;       // default: true
  };
  showToasts: {
    low_stock: boolean;     // default: true
    out_of_stock: boolean;  // default: true
    overstock: boolean;     // default: false
    reorder: boolean;       // default: true
  };
}
```

---

## Responsive Behavior

| Screen Size | Layout |
|---|---|
| Desktop (`lg:`) | Full alert list with all details and inline action buttons |
| Tablet (`md:`) | Same layout, slightly more compact |
| Mobile (default) | Alert cards stacked, actions in dropdown menu, swipe to dismiss |

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Same product triggers both low_stock and out_of_stock | Only `out_of_stock` created (quantity === 0 takes precedence) |
| Product deleted | All alerts for that product hard-deleted |
| Alert for product with changed name | Alert stores `productId`, name resolved at render time |
| 100+ alerts | Paginate or virtual scroll the alert list |
| Alert dismissed then condition recurs | New alert created (dismissed alert not reactivated) |
| All alert types disabled in settings | No new alerts created, existing alerts still visible |
| Restock resolves low_stock but creates overstock | Alert engine clears low_stock and creates overstock in same evaluation |

---

## Accessibility

- Alert cards use semantic `role="alert"` for critical alerts only
- Non-critical alerts use standard card markup (avoid alert fatigue for screen readers)
- Toast notifications use `aria-live="polite"`
- Alert icons have `aria-label` describing the severity
- Dismiss and Mark Read buttons have descriptive labels
- Color is supplemented with icons (not sole indicator)

---

## Test Scenarios

| Test | Type |
|------|------|
| Low stock alert created when quantity <= minStockLevel | Unit |
| Out of stock alert created when quantity === 0 | Unit |
| Overstock alert created when quantity > maxStockLevel | Unit |
| No duplicate alerts for same product + type | Unit |
| Alert auto-cleared when product restocked | Integration |
| Dismissed alerts hidden from default view | Component |
| Filter by type shows correct alerts | Component |
| Bulk "Mark All as Read" updates all unread alerts | Component |
| Toast shown when new alert created | Integration |
| Alert badge count is correct | Component |
| Alert settings toggle suppresses alert creation | Integration |
| Full alert lifecycle: create -> read -> dismiss | E2E |
