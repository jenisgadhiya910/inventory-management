# Charts & Analytics (Inventory Insights)

## What It Does

The charts and analytics section provides visual understanding of inventory performance. Interactive charts show trends, comparisons, and breakdowns to help users make smarter restocking decisions, identify top-performing products, and spot problems early.

---

## Component Architecture

```
features/analytics/
├── components/
│   ├── AnalyticsPage.tsx          # Full analytics page (/analytics route)
│   ├── AnalyticsSummaryCards.tsx   # Summary metric cards at top
│   ├── DateRangePicker.tsx        # Global date range selector
│   ├── ChartContainer.tsx         # Reusable wrapper: title, filters, export menu, error boundary
│   ├── StockTrendChart.tsx        # Chart 1: Line chart
│   ├── InventoryValueChart.tsx    # Chart 2: Area chart
│   ├── CategoryDistChart.tsx      # Chart 3: Pie/donut chart
│   ├── TopSellersChart.tsx        # Chart 4: Horizontal bar chart
│   ├── StockBreakdownChart.tsx    # Chart 5: Stacked bar / pie chart
│   ├── CategoryValueChart.tsx     # Chart 6: Bar chart
│   ├── SupplierPerfChart.tsx      # Chart 7: Bar chart
│   ├── StockMovementChart.tsx     # Chart 8: Grouped bar chart
│   ├── ProfitMarginChart.tsx      # Chart 9: Histogram
│   ├── LowStockForecastChart.tsx  # Chart 10: Line chart with threshold
│   └── ChartEmptyState.tsx        # Empty state for charts with no data
├── hooks/
│   ├── useDateRange.ts            # Global date range state
│   ├── useChartData.ts            # Data aggregation for specific chart types
│   └── useChartExport.ts          # PNG/CSV export logic
└── utils/
    ├── aggregation.ts             # Time-series aggregation (daily, weekly, monthly)
    ├── calculations.ts            # Turnover rate, forecast, trend calculations
    └── chartConfig.ts             # Recharts color palette, tooltip formatters
```

### Component Hierarchy

```
<AnalyticsPage>
  <PageHeader title="Analytics" />
  <DateRangePicker value={dateRange} onChange={setDateRange} />
  <AnalyticsSummaryCards dateRange={dateRange} />

  <div className="grid grid-cols-1 gap-6 lg:grid-cols-2">
    <ChartContainer title="Stock Level Trend" filters={...}>
      <StockTrendChart dateRange={dateRange} />
    </ChartContainer>
    <ChartContainer title="Inventory Value" filters={...}>
      <InventoryValueChart dateRange={dateRange} />
    </ChartContainer>
    {/* ... remaining charts */}
  </div>
</AnalyticsPage>
```

---

## ChartContainer (Reusable Wrapper)

```typescript
interface ChartContainerProps {
  title: string;
  description?: string;
  filters?: React.ReactNode;         // Chart-specific filter controls
  onExportPNG?: () => void;
  onExportCSV?: () => void;
  children: React.ReactNode;
}
```

Every chart is wrapped in `ChartContainer` which provides:
- Title and optional description
- Chart-specific filter dropdowns (rendered via `filters` prop)
- Export menu (PNG, CSV)
- Error boundary fallback
- Loading skeleton while data computes

```typescript
<ErrorBoundary fallback={<ChartErrorFallback title={title} />}>
  <Card>
    <CardHeader>
      <div className="flex items-center justify-between">
        <CardTitle>{title}</CardTitle>
        <div className="flex gap-2">
          {filters}
          <DropdownMenu>
            <DropdownMenuTrigger asChild><Button variant="ghost" size="icon"><Download /></Button></DropdownMenuTrigger>
            <DropdownMenuContent>
              <DropdownMenuItem onClick={onExportPNG}>Export as PNG</DropdownMenuItem>
              <DropdownMenuItem onClick={onExportCSV}>Export as CSV</DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>
        </div>
      </div>
    </CardHeader>
    <CardContent className="h-[300px]">
      {children}
    </CardContent>
  </Card>
</ErrorBoundary>
```

---

## Charts

### 1. Stock Level Trend (Line Chart)

**Purpose:** Total inventory quantity over time.

```typescript
// Data shape
interface StockTrendDataPoint {
  date: string;        // 'YYYY-MM-DD' or 'YYYY-WW' or 'YYYY-MM'
  totalQuantity: number;
}
```

| Axis | Value |
|------|-------|
| X | Time (days/weeks/months — selectable) |
| Y | Total stock quantity |

**Filters:** Time range, category (all or specific)

**Recharts implementation:**
```typescript
<ResponsiveContainer width="100%" height="100%">
  <LineChart data={trendData}>
    <CartesianGrid strokeDasharray="3 3" />
    <XAxis dataKey="date" tickFormatter={formatDateLabel} />
    <YAxis />
    <Tooltip formatter={(val) => [`${val} units`, 'Total Stock']} />
    <Line type="monotone" dataKey="totalQuantity" stroke="var(--primary)" strokeWidth={2} dot={false} />
  </LineChart>
</ResponsiveContainer>
```

---

### 2. Inventory Value Over Time (Area Chart)

**Purpose:** Total monetary value of inventory over time.

```typescript
interface ValueDataPoint {
  date: string;
  totalValue: number;   // sum of (quantity * price) for all products
}
```

**Calculation:** `products.reduce((sum, p) => sum + p.quantity * p[valueType], 0)` where `valueType` is `costPrice` or `sellingPrice`.

**Filters:** Time range, value type (cost price / selling price)

---

### 3. Products by Category (Pie/Donut Chart)

**Purpose:** Product distribution across categories.

```typescript
interface CategoryDistDataPoint {
  name: string;          // category name
  value: number;         // product count or total value
  color: string;         // category.color
}
```

**Toggle:** Product count vs. inventory value

**Recharts:** `<PieChart>` with `<Pie>` using `innerRadius` for donut effect. Center label shows total.

---

### 4. Top Selling Products (Horizontal Bar Chart)

**Purpose:** Best-performing products by units sold.

```typescript
interface TopSellerDataPoint {
  name: string;
  unitsSold: number;
  revenue: number;       // unitsSold * sellingPrice
  category: string;
}
```

**Recharts implementation:** Use `<BarChart layout="vertical">` for horizontal bars:

```typescript
<ResponsiveContainer width="100%" height="100%">
  <BarChart layout="vertical" data={topSellersData}>
    <CartesianGrid strokeDasharray="3 3" />
    <XAxis type="number" />
    <YAxis type="category" dataKey="name" width={120} />
    <Tooltip formatter={(val) => [`${val} units`, 'Units Sold']} />
    <Bar dataKey="unitsSold" fill="var(--primary)" radius={[0, 4, 4, 0]} />
  </BarChart>
</ResponsiveContainer>
```

**Calculation:**
```typescript
const salesByProduct = transactions
  .filter(t => t.type === 'sale' && isWithinDateRange(t.createdAt, dateRange))
  .reduce((acc, t) => {
    acc[t.productId] = (acc[t.productId] || 0) + t.quantity;
    return acc;
  }, {} as Record<string, number>);

// Sort by quantity, take top 10
```

**Filters:** Time range, category

---

### 5. Stock Status Breakdown (Donut Chart)

**Purpose:** Snapshot of in-stock vs. low-stock vs. out-of-stock products.

```typescript
const breakdown = [
  { name: 'In Stock', value: inStockCount, fill: '#22c55e' },
  { name: 'Low Stock', value: lowStockCount, fill: '#f59e0b' },
  { name: 'Out of Stock', value: outOfStockCount, fill: '#ef4444' },
];
```

Use `<PieChart>` with `<Pie innerRadius={60} outerRadius={80}>` for donut effect. Center label shows total product count using a custom `<text>` SVG element. Real-time — no date range filter needed.

---

### 6. Category-wise Stock Value (Bar Chart)

**Purpose:** Total stock value per category.

```typescript
interface CategoryValueDataPoint {
  name: string;
  value: number;         // sum of (quantity * costPrice) for products in category
  color: string;
}
```

**Sorting options:** By value (highest/lowest), by name (A-Z), by product count.

---

### 7. Supplier Performance (Bar Chart)

**Purpose:** Compare suppliers by key metrics.

**Switchable metrics:**
- Products supplied: `supplier.productsSupplied.length`
- Avg lead time: `supplier.leadTimeDays`
- Rating: `supplier.rating`
- Total value: sum of (quantity * costPrice) for supplier's products

---

### 8. Stock Movement History (Grouped Bar Chart)

**Purpose:** Restocks vs. sales vs. adjustments over time.

```typescript
interface MovementDataPoint {
  date: string;
  restocks: number;      // sum of quantities for type='restock'
  sales: number;         // sum of quantities for type='sale'
  adjustments: number;   // sum of quantities for type='adjustment'
}
```

Colors: Blue = restocks, Green = sales, Gray = adjustments.

**Filters:** Time range, specific product.

---

### 9. Profit Margin Distribution (Bar Chart as Histogram)

**Purpose:** How margins are spread across products.

> **Note:** Recharts has no native Histogram. Implement as `<BarChart>` with pre-bucketed data from `features/analytics/utils/calculations.ts`.

```typescript
// Bucket products into margin ranges
const buckets = [
  { range: '< 0%', count: 0 },
  { range: '0-10%', count: 0 },
  { range: '10-20%', count: 0 },
  { range: '20-30%', count: 0 },
  { range: '30-40%', count: 0 },
  { range: '40-50%', count: 0 },
  { range: '> 50%', count: 0 },
];

products.forEach(p => {
  const margin = calculateProfitMargin(p.costPrice, p.sellingPrice);
  // Assign to appropriate bucket
});
```

Negative margin products (`< 0%` bucket) highlighted in red.

---

### 10. Low Stock Forecast (Line Chart with Threshold)

**Purpose:** Predict when products will run out based on sales trends.

```typescript
interface ForecastDataPoint {
  date: string;
  actualQuantity?: number;
  projectedQuantity?: number;
}
```

**Calculation:**
```typescript
function forecastStockout(product: Product, transactions: StockTransaction[]): Date | null {
  const recentSales = transactions
    .filter(t => t.productId === product.id && t.type === 'sale')
    .filter(t => isWithinInterval(parseISO(t.createdAt), { start: subDays(new Date(), 30), end: new Date() }));

  const dailySalesRate = recentSales.reduce((sum, t) => sum + t.quantity, 0) / 30;
  if (dailySalesRate === 0) return null;

  const daysUntilStockout = product.quantity / dailySalesRate;
  return addDays(new Date(), Math.ceil(daysUntilStockout));
}
```

Visual: solid line for actual, dashed line for projected. Horizontal dashed line at `minStockLevel`. Intersection point labeled with estimated date.

---

## Analytics Summary Cards

```typescript
interface AnalyticsSummary {
  totalInventoryValue: number;       // sum(quantity * costPrice)
  totalRetailValue: number;          // sum(quantity * sellingPrice)
  averageMargin: number;             // avg of all product margins
  turnoverRate: number;              // total units sold / average stock level (over period)
  stockoutFrequency: number;         // count of out_of_stock alerts created this month
  reorderEfficiency: number;         // % of reorders placed before stockout occurred
}
```

### Turnover Rate Calculation

```typescript
function calculateTurnoverRate(transactions: StockTransaction[], products: Product[], dateRange: DateRange): number {
  const totalSold = transactions
    .filter(t => t.type === 'sale' && isWithinDateRange(t.createdAt, dateRange))
    .reduce((sum, t) => sum + t.quantity, 0);
  const avgStock = products.reduce((sum, p) => sum + p.quantity, 0) / (products.length || 1);
  return avgStock > 0 ? totalSold / avgStock : 0;
}
```

### Reorder Efficiency Calculation

```typescript
function calculateReorderEfficiency(transactions: StockTransaction[], alerts: StockAlert[]): number {
  const restockCount = transactions.filter(t => t.type === 'restock').length;
  const stockoutCount = alerts.filter(a => a.type === 'out_of_stock').length;
  if (restockCount === 0) return 0;
  const preemptiveRestocks = restockCount - stockoutCount;
  return Math.max(0, (preemptiveRestocks / restockCount) * 100);
}
```

---

## Date Range Picker

```typescript
interface DateRange {
  start: Date;
  end: Date;
  preset: 'today' | '7d' | '30d' | '90d' | '12m' | 'custom';
}
```

**Presets:**

| Label | Start | End |
|-------|-------|-----|
| Today | startOfDay(now) | now |
| Last 7 days | subDays(now, 7) | now |
| Last 30 days | subDays(now, 30) | now |
| Last 90 days | subDays(now, 90) | now |
| Last 12 months | subMonths(now, 12) | now |
| Custom | User-selected start | User-selected end |

Global picker at top of analytics page. Individual charts can override with their own filter.

```typescript
// useDateRange.ts
const [dateRange, setDateRange] = useState<DateRange>({
  start: subDays(new Date(), 30),
  end: new Date(),
  preset: '30d',
});
```

---

## Chart Interactions (All Charts)

| Interaction | Implementation |
|-------------|---------------|
| Hover tooltip | Recharts `<Tooltip>` component with custom formatter |
| Click drill-down | `onClick` handler on chart elements (e.g., pie slice -> navigate to category) |
| Legend toggle | Recharts `<Legend>` with `onClick` to show/hide data series |
| Zoom (time-based) | Recharts `<Brush>` component for range selection |
| Reset zoom | Button appears after zoom, resets `<Brush>` to full range |
| Export PNG | Use `html2canvas` on chart container element |
| Export CSV | Build CSV string from chart data, trigger download via `Blob` + `URL.createObjectURL` |

---

## Data Aggregation

```typescript
// utils/aggregation.ts
type TimeGranularity = 'day' | 'week' | 'month';

function aggregateByTime<T>(
  data: T[],
  getDate: (item: T) => string,
  getValue: (item: T) => number,
  granularity: TimeGranularity
): { date: string; value: number }[] {
  const buckets = new Map<string, number>();

  data.forEach(item => {
    const date = parseISO(getDate(item));
    const key = granularity === 'day' ? format(date, 'yyyy-MM-dd')
              : granularity === 'week' ? format(date, "yyyy-'W'II")
              : format(date, 'yyyy-MM');
    buckets.set(key, (buckets.get(key) || 0) + getValue(item));
  });

  return Array.from(buckets.entries())
    .map(([date, value]) => ({ date, value }))
    .sort((a, b) => a.date.localeCompare(b.date));
}
```

---

## Empty States

| Situation | Message | Action |
|-----------|---------|--------|
| No products | "Add products to start seeing analytics." | Button -> `/products/new` |
| No transactions | "Stock movement charts will appear after your first restock or sale." | None |
| No data for date range | "No data available for this time period. Try selecting a wider date range." | None |

---

## Responsive Behavior

| Screen Size | Layout |
|---|---|
| Desktop (`lg:`) | 2-column grid, full interactivity |
| Tablet (`md:`) | Single column, full interactivity |
| Mobile (default) | Single column, simplified tooltips, swipe through charts |

```typescript
<div className="grid grid-cols-1 gap-6 lg:grid-cols-2">
  {charts.map(chart => <ChartContainer key={chart.id} {...chart} />)}
</div>
```

---

## Performance

| Concern | Strategy |
|---------|----------|
| Expensive aggregations | `useMemo` with `[transactions, products, dateRange]` dependencies |
| Multiple charts re-rendering | Each chart memoized; only re-renders when its specific data changes |
| Large transaction history | Limit to last 10,000 transactions; aggregate older data |
| Chart rendering | Recharts `<ResponsiveContainer>` handles resize. Use `isAnimationActive={false}` for large datasets |
| Export PNG | Async operation with loading indicator |

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| No sales transactions | Top sellers chart shows empty state, forecast returns null |
| Zero cost price | Profit margin shows 100% (formula: `(selling - 0) / selling * 100`) |
| All products same category | Pie chart shows single slice at 100% |
| Date range with no data | All charts show "No data for this period" empty state |
| Negative margin products | Histogram highlights red bar, tooltip shows "selling below cost" |
| Division by zero in turnover | Return 0, display "N/A" |
| Forecast with no sales history | Show "Insufficient data for forecast" message |

---

## Accessibility

- All charts have descriptive titles and labels readable by screen readers
- Color choices meet WCAG contrast requirements
- Patterns/labels supplement color coding (not color-only)
- Data table toggle: every chart has an alternative table view (`<Table>` with same data)
- Keyboard navigation for chart elements via arrow keys
- Tooltips accessible via keyboard focus (Tab to chart element)
- `aria-label` on chart containers describing the visualization

---

## Test Scenarios

| Test | Type |
|------|------|
| Summary cards calculate correct values | Unit |
| Turnover rate handles zero average stock | Unit |
| Stock trend aggregation groups by day/week/month | Unit |
| Profit margin histogram buckets products correctly | Unit |
| Forecast calculates correct stockout date | Unit |
| Date range picker updates all charts | Component |
| Chart empty state renders when no data | Component |
| Export CSV generates valid file | Unit |
| Responsive layout switches from 2-col to 1-col | Component |
| Full analytics page renders without errors | E2E |
