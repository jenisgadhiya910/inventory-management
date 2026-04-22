# Charts & Analytics (Inventory Insights)

## What It Does

The charts and analytics section gives you a visual understanding of your inventory performance over time. Instead of reading raw numbers, you can see trends, comparisons, and breakdowns through interactive charts. This helps you make smarter restocking decisions, identify top-performing products, and spot problems early.

---

## Where It Lives

- **Dashboard** — Key charts appear below the stats cards on the main dashboard
- **Analytics Page** — A dedicated page (`/analytics`) with the full set of charts and reports
- **Product Detail** — Individual product pages show that product's stock history chart

---

## Charts Overview

### 1. Stock Level Trend (Line Chart)

**What it shows:** How your total inventory quantity has changed over time.

- X-axis: Time (days, weeks, or months — selectable)
- Y-axis: Total stock quantity
- A single line showing overall stock movement
- Hover over any point to see the exact quantity and date
- Useful for spotting seasonal patterns or gradual stock decline

**Filters:**
- Time range: Last 7 days, 30 days, 90 days, 12 months, or custom range
- Category: View trend for a specific category or all categories

---

### 2. Inventory Value Over Time (Area Chart)

**What it shows:** The total monetary value of your inventory over time.

- X-axis: Time (days, weeks, or months)
- Y-axis: Total value (quantity x cost price for all products)
- Filled area under the line for visual emphasis
- Hover to see the exact value at any point
- Helps you understand if your capital tied up in inventory is growing or shrinking

**Filters:**
- Time range: Last 7 days, 30 days, 90 days, 12 months, or custom range
- Value type: By cost price or by selling price

---

### 3. Products by Category (Pie / Donut Chart)

**What it shows:** How your products are distributed across categories.

- Each slice represents one category, sized by the number of products in it
- Category color matches the color set in category settings
- Hover over a slice to see the category name, product count, and percentage
- Click a slice to navigate to that category's product listing
- Center of the donut shows the total product count

**Toggle:** Switch between showing distribution by product count or by inventory value

---

### 4. Top Selling Products (Horizontal Bar Chart)

**What it shows:** Your best-performing products based on stock movement (sales transactions).

- Shows the top 10 products by number of units sold
- Each bar displays the product name and total units sold
- Bars are color-coded by category
- Hover for details: product name, units sold, revenue generated
- Click a bar to open that product's detail view

**Filters:**
- Time range: Last 7 days, 30 days, 90 days, or 12 months
- Category: Filter by specific category

---

### 5. Stock Status Breakdown (Stacked Bar / Pie Chart)

**What it shows:** A snapshot of how many products are in stock, low stock, or out of stock.

- Three segments: In Stock (green), Low Stock (amber), Out of Stock (red)
- Shows both count and percentage for each segment
- Updates in real-time as stock levels change
- Useful as a quick health check for your entire inventory

---

### 6. Category-wise Stock Value (Bar Chart)

**What it shows:** The total stock value held in each category.

- X-axis: Category names
- Y-axis: Total value (quantity x cost price for all products in that category)
- Bars colored by category color
- Sorted by value (highest first) by default
- Hover for exact value and product count

**Sorting options:**
- By value (highest/lowest)
- By name (A-Z)
- By product count

---

### 7. Supplier Performance (Bar Chart)

**What it shows:** A comparison of your suppliers based on key metrics.

- Each bar represents one supplier
- Metrics you can switch between:
  - Number of products supplied
  - Average lead time (days)
  - Supplier rating
  - Total stock value supplied
- Helps identify your most reliable and valuable suppliers

---

### 8. Stock Movement History (Grouped Bar Chart)

**What it shows:** Restocks vs. sales vs. adjustments over time.

- X-axis: Time periods (days, weeks, or months)
- Y-axis: Quantity of items
- Three grouped bars per period:
  - Blue = Restocks (stock added)
  - Green = Sales (stock removed)
  - Gray = Adjustments (manual corrections)
- Hover for exact quantities per transaction type
- Useful for understanding your inventory flow rate

**Filters:**
- Time range: Last 7 days, 30 days, 90 days, or 12 months
- Product: View movement for a specific product

---

### 9. Profit Margin Distribution (Histogram / Bar Chart)

**What it shows:** How profit margins are spread across your products.

- X-axis: Margin ranges (e.g., 0-10%, 10-20%, 20-30%, etc.)
- Y-axis: Number of products in each range
- Helps you see if most products have healthy margins or if many are underpriced
- Products with negative margins (selling below cost) are highlighted in red

---

### 10. Low Stock Forecast (Line Chart with Threshold)

**What it shows:** A projection of when products are likely to run out of stock based on recent sales trends.

- Shows current stock level and a projected decline line
- A horizontal dashed line marks the minimum stock level (reorder point)
- The intersection point indicates the estimated stockout date
- Products predicted to run out within 7 days are highlighted
- Available per product or as a summary showing the 5 most urgent items

---

## Analytics Summary Cards

At the top of the analytics page, summary cards provide key metrics:

| Metric | Description |
|---|---|
| Total Inventory Value | Sum of (quantity x cost price) for all products |
| Total Retail Value | Sum of (quantity x selling price) for all products |
| Average Margin | Average profit margin across all products |
| Turnover Rate | Average rate at which stock is sold and replaced |
| Stockout Frequency | Number of times products went out of stock this month |
| Reorder Efficiency | Percentage of reorders placed before stockout |

---

## Chart Interactions

All charts share these common interactions:

- **Hover** — Tooltip appears with exact values
- **Click** — Drill down into the data (e.g., click a category slice to see its products)
- **Legend** — Click a legend item to show/hide that data series
- **Zoom** — On time-based charts, click and drag to zoom into a time range
- **Reset** — A "Reset Zoom" button appears after zooming
- **Export** — Each chart has a menu to download as PNG or export the underlying data as CSV

---

## Date Range Picker

A global date range picker at the top of the analytics page sets the time range for all charts at once. Options include:

- Today
- Last 7 days
- Last 30 days
- Last 90 days
- Last 12 months
- Custom range (calendar picker for start and end dates)

Individual charts can also override the global range with their own filter.

---

## Data Source

All analytics data is computed from:

- **Products** — Current stock levels, prices, and categories
- **Stock Transactions** — Historical restocks, sales, adjustments, and returns
- **Categories & Suppliers** — For grouping and comparison charts

Since the app uses localStorage, all chart data is calculated on the client side in real-time. There is no separate analytics database.

---

## Empty States

| Situation | What you see |
|---|---|
| No products added yet | "Add products to start seeing analytics." with an "Add Product" button |
| No transactions yet | "Stock movement charts will appear after your first restock or sale." |
| No data for selected date range | "No data available for this time period. Try selecting a wider date range." |

---

## Responsive Behavior

| Screen Size | What happens |
|---|---|
| Desktop | Charts displayed in a 2-column grid layout, full interactivity |
| Tablet | Charts displayed in a single column, full interactivity |
| Mobile | Charts stack vertically, simplified tooltips, swipe to scroll through charts |

---

## Accessibility

- All charts include descriptive labels and titles readable by screen readers
- Color choices meet contrast requirements; patterns or labels supplement color coding
- Data tables are available as an alternative view for every chart (toggle between chart and table)
- Keyboard users can navigate chart elements using arrow keys
- Tooltips are accessible via keyboard focus, not just hover
