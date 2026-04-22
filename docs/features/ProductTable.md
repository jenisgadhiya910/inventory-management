# Product Table (Product Listing)

## What It Does

The product table is the main view for browsing and managing all products in your inventory. It presents products in a spreadsheet-like format with sorting, filtering, searching, and bulk actions.

---

## Table Layout

Each row represents one product. The columns are:

- **Checkbox** — Select one or more products for bulk actions
- **Image** — A small thumbnail of the product
- **Product Name** — The name of the product (click to open details)
- **SKU** — The unique product code
- **Category** — Which category the product belongs to
- **Stock Quantity** — How many units are in stock
- **Cost Price** — What you pay for the product
- **Selling Price** — What you sell it for
- **Status** — In Stock, Low Stock, or Out of Stock (color-coded)
- **Actions** — A three-dot menu with more options

---

## Sorting

- Click any column header to sort by that column
- Click again to reverse the sort order
- Click a third time to clear the sort
- An arrow icon shows which column is being sorted and in which direction
- Default sort: newest products first

---

## Filtering

A toolbar above the table lets you filter products:

- **Search** — Real-time search across product names, SKUs, and descriptions
- **Category** — Show only products in selected categories
- **Supplier** — Show products from specific suppliers
- **Status** — Filter by In Stock, Low Stock, or Out of Stock
- **Price Range** — Set minimum and maximum price values

Active filters appear as removable chips below the filter bar, with a "Clear all filters" option.

---

## Row Interactions

- **Click product name** — Opens the full product details
- **Click checkbox** — Selects the product for bulk actions
- **Click status badge** — Lets you quickly toggle in-stock / out-of-stock
- **Click three-dot menu** — Options include Edit, Restock, Duplicate, Archive, Delete

---

## Bulk Actions

When you select one or more products, a toolbar appears:

- **Change Category** — Move all selected products to a chosen category
- **Update Status** — Mark selected products as in-stock or out-of-stock
- **Export** — Download selected products as a file
- **Delete** — Delete all selected products (asks for confirmation)

You can also use the header checkbox to select all visible products at once.

---

## Pagination

- Shows 10 products per page by default
- You can change this to 25, 50, or All
- Previous/Next buttons and page numbers let you navigate
- A label shows your position (e.g., "Showing 1-10 of 156 products")

---

## Stock Level Indicators

Each product row shows stock status visually:

| Status | How it looks |
|---|---|
| In Stock | Green badge — quantity is above minimum level |
| Low Stock | Amber/orange badge — quantity is at or below minimum level |
| Out of Stock | Red badge — quantity is zero |

A small progress bar next to the quantity shows how full the stock is relative to the maximum level.

---

## Inline Editing

You can edit certain fields directly in the table without opening the product:

- **Quantity** — Click to adjust stock with a quick +/- control
- **Selling Price** — Click to type a new price
- **Status** — Click to toggle in-stock / out-of-stock

Changes are saved automatically.

---

## Empty States

| Situation | What you see |
|---|---|
| No products added yet | "No products yet. Add your first product to get started." with a button |
| No products match filters | "No products match your filters." with a "Clear filters" link |
| All products archived | "All products are archived. Toggle 'Show archived' to view them." |

---

## Responsive Behavior

| Screen Size | What happens |
|---|---|
| Desktop | Full table with all columns |
| Tablet | Some columns hidden, horizontal scrolling available |
| Mobile | Switches to a card-based stacked layout instead of a table |
