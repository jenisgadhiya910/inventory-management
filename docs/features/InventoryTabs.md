# Inventory Tabs (Products, Categories, Suppliers, Alerts)

## What It Does

Tabs let you switch between four main sections of the inventory system. Each tab shows different data and serves a different purpose.

---

## The Four Tabs

### Products Tab (Default)
- The main product listing table with search, sort, and filter
- Where you manage all your inventory items
- See the ProductTable document for full details

### Categories Tab
- Browse and manage product categories
- Products are grouped under their categories using expandable sections
- See the CategoryAccordion document for full details

### Suppliers Tab
- View and manage all your suppliers
- See supplier details, ratings, and which products they supply

### Alerts Tab
- View all stock-related notifications and warnings
- See the StockAlerts document for full details

---

## Products Tab Details

- Shows the full product data table
- Quick access to "Add Product" button
- Filter and search tools are always visible
- Product count is shown in a badge next to the tab label

---

## Categories Tab Details

### Category List
- Each category shows its name, description, color, and product count
- Categories can be expanded to see all products within them
- Supports parent/child category relationships (subcategories)

### Category Management
- **Add Category** button at the top
- Each category has options to: Edit, Add Subcategory, Delete
- Deleting a category asks for confirmation and requires you to reassign its products first
- Drag to reorder categories (or use the sort dropdown)

---

## Suppliers Tab Details

### Supplier List
- Displayed as a table with columns:
  - Supplier name and company
  - Contact info (email, phone)
  - Lead time (how many days deliveries take)
  - Rating (shown as stars)
  - Number of products supplied
  - Active/Inactive status
- Click a supplier name to view full details

### Supplier Management
- **Add Supplier** button at the top
- Each supplier has options to: Edit, Deactivate, Delete
- Deleting a supplier asks for confirmation
- Filter by active/inactive status

---

## Alerts Tab Details

- Displays all stock alerts sorted by severity (critical first)
- Each alert shows:
  - Alert type icon and color
  - Product name
  - Alert message (e.g., "Widget X is below minimum stock level")
  - When the alert was created
  - Mark as Read / Dismiss buttons
- Filter alerts by: type, severity, read/unread status
- "Mark All as Read" and "Dismiss All" buttons at the top
- Alert count badge on the tab shows unread alerts

---

## Tab Behavior

- The active tab is reflected in the page URL, so you can bookmark or share a specific section
- Your last active tab is remembered when you return
- Each tab maintains its own filter and search state independently
- Use arrow keys to navigate between tabs when focused on the tab bar

---

## Badge Counts on Tabs

Each tab shows a small count badge:

- **Products** — Total number of products
- **Categories** — Total number of categories
- **Suppliers** — Total number of active suppliers
- **Alerts** — Number of unread alerts (highlighted in red if any are critical)

---

## Responsive Behavior

| Screen Size | What happens |
|---|---|
| Desktop | Full tab bar with icons and labels |
| Tablet | Icons and labels, slightly smaller |
| Mobile | Icons only (labels hidden to save space) |
