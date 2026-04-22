# Stock Alerts (Low Stock Warnings & Notifications)

## What It Does

Stock alerts notify you when products need attention — whether they're running low, completely out of stock, overstocked, or approaching expiry. Alerts appear as badges, banners, and notifications throughout the app so you never miss a critical inventory issue.

---

## Alert Types

### 1. Low Stock Alert
- **Triggers when:** A product's quantity drops to or below its minimum stock level
- **Severity:** Warning (amber/orange)
- **Message example:** "Widget X has only 3 units left (minimum: 10)"

### 2. Out of Stock Alert
- **Triggers when:** A product's quantity reaches zero
- **Severity:** Critical (red)
- **Message example:** "Widget X is out of stock"

### 3. Overstock Alert
- **Triggers when:** A product's quantity exceeds its maximum stock level
- **Severity:** Info (blue)
- **Message example:** "Widget X has 150 units (maximum: 100)"

### 4. Reorder Reminder
- **Triggers when:** Stock is low AND the supplier's lead time means you should order now to avoid running out
- **Severity:** Warning (amber/orange)
- **Message example:** "Order Widget X now — supplier lead time is 7 days and current stock will last approximately 5 days"

---

## Where Alerts Appear

### Dashboard Banner
- Critical alerts (out of stock) appear as a red banner at the top of the dashboard
- Shows the count and a "View All" link

### Alert Badge on Navigation
- A small red badge with the unread alert count appears on the Alerts tab
- Also shown on the bell icon in the app header

### Product Table
- Products with active alerts show a warning icon next to their name
- The status column shows the appropriate colored badge

### Toast Notifications
- When a new alert is triggered (e.g., after a sale reduces stock), a brief notification pops up in the corner
- Shows the alert message with a "View" link

### Alerts Tab
- The dedicated alerts page (see Inventory Tabs document) shows all alerts in a full list

---

## Alert List Layout

Each alert in the list shows:

- **Severity icon** — Color-coded icon (red for critical, amber for warning, blue for info)
- **Product name** — Which product the alert is about (clickable to open product detail)
- **Alert message** — What happened and why
- **Time** — When the alert was created (e.g., "2 hours ago")
- **Actions:**
  - **Restock** — Quick link to open the restock dialog for that product
  - **Mark as Read** — Removes the unread indicator
  - **Dismiss** — Hides the alert from the list

---

## Alert Management

### Filtering alerts
- **By type** — Low Stock, Out of Stock, Overstock, Reorder Reminder
- **By severity** — Critical, Warning, Info
- **By status** — Unread, Read, Dismissed

### Bulk actions
- **Mark All as Read** — Clears all unread indicators
- **Dismiss All** — Hides all alerts (with confirmation)

### Alert settings (in the Settings page)
- Toggle each alert type on or off
- Set custom thresholds for low stock (override per-product minimums with a global default)
- Choose whether toast notifications appear for each alert type

---

## Alert Lifecycle

1. Alert is created automatically when a condition is met
2. Alert appears as unread with a bold indicator
3. User views or marks the alert as read
4. User can dismiss the alert to hide it
5. If the condition is resolved (e.g., product is restocked), the alert is automatically cleared

---

## Alert Counts

| Location | What it shows |
|---|---|
| App header bell icon | Total unread alerts |
| Alerts tab badge | Total unread alerts (red if any are critical) |
| Dashboard banner | Count of critical (out of stock) alerts only |
| Stats card (Low Stock) | Count of low-stock products |

---

## Responsive Behavior

| Screen Size | What happens |
|---|---|
| Desktop | Full alert list with all details visible |
| Tablet | Same layout, slightly more compact |
| Mobile | Alerts stack as cards, swipe to dismiss |
