# Stock Toggles (In-Stock, Featured, Settings)

## What It Does

Toggles are simple on/off switches used throughout the app. They let you mark products as in-stock or out-of-stock, highlight featured items, and manage system settings. Every toggle saves your choice immediately.

---

## Toggle Types

### 1. In-Stock / Out-of-Stock Toggle

**Where:** Product table rows, product detail panel, edit product dialog

- **On (In Stock)** — Product is available and visible in normal listings
- **Off (Out of Stock)** — Product is marked as unavailable
- When you mark a product out of stock:
  - The status badge changes to red "Out of Stock"
  - The product row becomes slightly faded in the table
  - An "Out of Stock" alert is automatically created
  - A notification message appears with an "Undo" option (available for 5 seconds)
- When you toggle it back to in stock, everything returns to normal
- Note: This is a manual override — stock status also updates automatically when quantity reaches zero

---

### 2. Featured Item Toggle

**Where:** Product detail panel, edit product dialog, product table

- **On** — Product is marked as featured and gets a star icon
- **Off** — Product is a regular item
- Featured products appear at the top of the product listing when sorted by "Featured first"
- Useful for highlighting bestsellers, new arrivals, or promoted items

---

### 3. Supplier Active Toggle

**Where:** Supplier detail panel, supplier table

- **On** — Supplier is active and available for selection when adding products
- **Off** — Supplier is inactive and hidden from the supplier dropdown in product forms
- Deactivating a supplier does not affect products already linked to them

---

### 4. Dark Mode Toggle

**Where:** App header menu, settings

- **Off** — Light mode (or follows your system preference)
- **On** — Dark mode
- The switch appears between a sun icon and a moon icon
- Changes apply instantly across the entire app

---

### 5. Alert Notification Toggles

**Where:** Settings page

Each alert type has its own toggle:

- **Low Stock Alerts** — Get notified when a product drops below its minimum stock level
- **Out of Stock Alerts** — Get notified when a product reaches zero quantity
- **Overstock Alerts** — Get notified when a product exceeds its maximum stock level
- **Reorder Reminders** — Get notified when it's time to reorder based on supplier lead time

---

## Filter Toggles

**Where:** Product table filter bar

Quick-toggle buttons for common filters:

- **Low Stock Only** — Show only products that need restocking
- **Featured Only** — Show only featured products
- **Show Archived** — Include archived products in the listing

These appear as toggle buttons that light up when active.

---

## What Gets Saved Where

| Toggle | Saved permanently? |
|---|---|
| In-stock / Out-of-stock | Yes |
| Featured item | Yes |
| Supplier active | Yes |
| Dark mode | Yes |
| Alert notifications | Yes |
| Filter toggles | No (resets when you leave the page) |

---

## Accessibility

- All toggles can be controlled with keyboard (Space or Enter to toggle)
- Each toggle has a visible label
- Screen readers announce the current state (on/off)
- Focus outlines are visible when navigating with keyboard
