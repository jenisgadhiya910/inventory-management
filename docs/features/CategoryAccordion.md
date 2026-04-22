# Category Accordion (Category-wise Product Grouping)

## What It Does

The category accordion organizes products into expandable/collapsible groups based on their category. Instead of scrolling through a flat list, you can open one category at a time to see only the products that belong to it.

---

## How It Looks

- Each category appears as a collapsible row/section
- Click a category to expand it and reveal the products inside
- Click again to collapse it
- Only one category can be open at a time (or you can configure it to allow multiple)

---

## Category Header (Collapsed)

Each category row shows:

- **Category color** — A small colored dot or left border matching the category color
- **Category name** — The name of the category (e.g., "Electronics")
- **Product count** — A badge showing how many products are in this category (e.g., "24 products")
- **Stock health summary** — Small indicators:
  - Green dot + count = in-stock products
  - Amber dot + count = low-stock products
  - Red dot + count = out-of-stock products
- **Expand/Collapse arrow** — Chevron icon that rotates when opened

---

## Category Content (Expanded)

When you expand a category, you see:

### Category Info Bar
- Category description
- Edit Category and Delete Category buttons

### Product List
- A mini table or card list showing all products in that category
- Each product row shows:
  - Product name
  - SKU
  - Stock quantity with status badge (In Stock / Low Stock / Out of Stock)
  - Selling price
  - Quick actions: Edit, Restock, View Details

### Empty Category
- If a category has no products: "No products in this category yet. Add a product to get started."

---

## Subcategories

- Categories can have subcategories (e.g., "Electronics" > "Phones", "Electronics" > "Laptops")
- Subcategories appear nested inside their parent category when expanded
- Each subcategory is also collapsible with the same layout
- Indent level visually shows the parent/child relationship

---

## Category Actions

### From the category header
- **Right-click** or **three-dot menu** on a category for options:
  - Edit Category
  - Add Subcategory
  - Add Product to Category
  - Move All Products (to another category)
  - Delete Category

### Deleting a category
- A confirmation dialog appears
- You must choose what to do with its products: move to another category or delete them
- Cannot delete a category that has subcategories without handling them first

---

## Sorting Categories

You can sort categories by:
- **Name** (A-Z or Z-A)
- **Product count** (most to fewest, or fewest to most)
- **Recently updated**
- **Custom order** (drag to reorder)

---

## Searching Within Categories

- A search bar at the top filters categories by name
- When searching, only matching categories are shown, and they auto-expand to reveal matching products
- Search also matches product names within categories

---

## Responsive Behavior

| Screen Size | What happens |
|---|---|
| Desktop | Full accordion with product mini-tables inside |
| Tablet | Same layout, slightly more compact |
| Mobile | Accordion with product cards stacked vertically inside each section |

---

## Accessibility

- Each accordion section can be opened/closed with keyboard (Enter or Space)
- Arrow keys navigate between category headers
- Screen readers announce "expanded" or "collapsed" state
- Focus moves into the expanded content when a section opens
