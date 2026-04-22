# Category Accordion (Category-wise Product Grouping)

## What It Does

The category accordion organizes products into expandable/collapsible groups based on their category. Users can open one category at a time to see only the products that belong to it, manage subcategories, and perform category-level actions.

> **Dependencies:** Reads from `CategoryContext` and `ProductContext` (products filtered by `categoryId`). Writes to `CategoryContext` (CRUD), `ProductContext` (move products between categories). "Add Product to Category" navigates to [AddProductWizard](AddProductWizard.md) with `?categoryId=` pre-fill. Product actions (Edit, Restock, View) open [ProductDialog](ProductDialog.md).

---

## Component Architecture

```
features/categories/
├── components/
│   ├── CategoryAccordion.tsx     # Container: renders list of CategoryItem components
│   ├── CategoryItem.tsx          # Single accordion item (header + collapsible content)
│   ├── CategoryHeader.tsx        # Color dot, name, count, health indicators, expand arrow
│   ├── CategoryContent.tsx       # Expanded content: info bar + product list
│   ├── CategoryProductList.tsx   # Mini table/cards of products in this category
│   ├── CategoryActions.tsx       # Context menu: edit, add subcategory, delete, etc.
│   ├── CategoryForm.tsx          # Add/edit category dialog form
│   └── CategorySearch.tsx        # Search bar that filters and auto-expands
├── hooks/
│   ├── useCategories.ts          # CRUD operations via CategoryContext
│   ├── useCategoryTree.ts        # Builds parent/child tree structure
│   └── useCategorySort.ts        # Sort state for category list
└── context/
    └── CategoryContext.tsx        # State + reducer for categories
```

### Component Hierarchy

```
<CategoryManagement>               # Route component at /categories
  <PageHeader title="Categories" action={<Button>Add Category</Button>} />
  <CategorySearch onSearch={setSearchTerm} />
  <CategorySortDropdown value={sort} onChange={setSort} />
  <CategoryAccordion>
    {categoryTree.map(category => (
      <CategoryItem
        key={category.id}
        category={category}
        products={productsByCategory[category.id]}
        expanded={expandedId === category.id}
        onToggle={handleToggle}
      >
        {category.children?.map(sub => (
          <CategoryItem                    {/* Nested subcategory */}
            key={sub.id}
            category={sub}
            products={productsByCategory[sub.id]}
            depth={1}
          />
        ))}
      </CategoryItem>
    ))}
  </CategoryAccordion>
</CategoryManagement>
```

---

## Category Header (Collapsed View)

```typescript
interface CategoryHeaderProps {
  category: Category;
  productCount: number;
  stockHealth: {
    inStock: number;
    lowStock: number;
    outOfStock: number;
  };
  expanded: boolean;
  depth: number;       // 0 = top-level, 1 = subcategory
}
```

### Visual Elements

| Element | Implementation |
|---------|---------------|
| Color dot | `<div className="h-3 w-3 rounded-full" style={{ backgroundColor: category.color }} />` |
| Category name | `<span className="font-medium">{category.name}</span>` |
| Product count | `<Badge variant="secondary">{count} products</Badge>` |
| Stock health | Green/amber/red dots with counts: `<span className="text-green-600">{inStock}</span>` |
| Expand arrow | `<ChevronDown className={cn("transition-transform", expanded && "rotate-180")} />` |
| Indent | `style={{ paddingLeft: depth * 24 }}` for subcategories |

---

## Category Content (Expanded View)

### Info Bar

```
<div className="flex items-center justify-between border-b p-3">
  <p className="text-sm text-muted-foreground">{category.description}</p>
  <div className="flex gap-2">
    <Button size="sm" variant="outline" onClick={openEditDialog}>Edit</Button>
    <Button size="sm" variant="destructive" onClick={openDeleteConfirm}>Delete</Button>
  </div>
</div>
```

### Product List (Mini Table)

| Column | Width | Notes |
|--------|-------|-------|
| Product Name | flex | Click -> product detail dialog |
| SKU | 100px | Monospace |
| Quantity + Status Badge | 120px | Same badge logic as ProductTable |
| Selling Price | 100px | Formatted currency |
| Actions | 80px | Edit / Restock / View buttons |

```typescript
// Get products for a category
const categoryProducts = useMemo(
  () => products.filter(p => p.categoryId === category.id),
  [products, category.id]
);
```

### Empty Category

```typescript
{categoryProducts.length === 0 && (
  <EmptyState
    message="No products in this category yet."
    action={<Button size="sm" onClick={() => navigate(`/products/new?categoryId=${category.id}`)}>Add Product</Button>}
  />
)}
```

---

## Expand/Collapse Behavior

### Single Expand Mode (Default)

```typescript
const [expandedId, setExpandedId] = useState<string | null>(null);

const handleToggle = (categoryId: string) => {
  setExpandedId(prev => prev === categoryId ? null : categoryId);
};
```

Only one category open at a time. Opening one closes the other. This keeps the page manageable.

### Accordion Animation

Use shadcn `Accordion` component which uses Radix Accordion with built-in enter/exit animations:

```typescript
<Accordion type="single" collapsible value={expandedId} onValueChange={setExpandedId}>
  {categories.map(cat => (
    <AccordionItem key={cat.id} value={cat.id}>
      <AccordionTrigger><CategoryHeader ... /></AccordionTrigger>
      <AccordionContent><CategoryContent ... /></AccordionContent>
    </AccordionItem>
  ))}
</Accordion>
```

---

## Subcategories

### Tree Building

```typescript
// hooks/useCategoryTree.ts
interface CategoryTreeNode extends Category {
  children: CategoryTreeNode[];
}

function buildCategoryTree(categories: Category[]): CategoryTreeNode[] {
  const map = new Map<string, CategoryTreeNode>();
  const roots: CategoryTreeNode[] = [];

  // First pass: create nodes
  categories.forEach(cat => map.set(cat.id, { ...cat, children: [] }));

  // Second pass: link parents to children
  categories.forEach(cat => {
    const node = map.get(cat.id)!;
    if (cat.parentId && map.has(cat.parentId)) {
      map.get(cat.parentId)!.children.push(node);
    } else {
      roots.push(node);
    }
  });

  return roots;
}
```

### Nesting Rules

- Maximum depth: 2 levels (top-level + one subcategory level). The "Add Subcategory" action is hidden on depth-1 categories.
- Subcategories inherit parent's color if not explicitly set
- Subcategory indented by 24px per depth level
- Subcategories are collapsible independently when parent is expanded
- A subcategory's `parentId` **can** be changed via Edit Category dialog (re-parenting). Setting `parentId` to `null` promotes it to top-level.
- Category `productCount` is a denormalized counter. It must be updated whenever: a product is added to the category, moved out, or deleted.

---

## Category Actions (Context Menu)

```typescript
// CategoryActions.tsx — triggered by three-dot menu or right-click
interface CategoryActionsProps {
  category: Category;
  onEdit: () => void;
  onAddSubcategory: () => void;
  onAddProduct: () => void;
  onMoveProducts: () => void;
  onDelete: () => void;
}
```

| Action | Behavior |
|--------|----------|
| Edit Category | Opens `CategoryForm` dialog pre-filled |
| Add Subcategory | Opens `CategoryForm` dialog with `parentId` set |
| Add Product to Category | Navigates to `/products/new?categoryId={id}` |
| Move All Products | Opens category select dialog, moves products to chosen category |
| Delete Category | Opens confirmation with product reassignment |

### Delete Category Flow

```
User clicks Delete
  -> if category has subcategories:
     -> Show error: "Remove or reassign subcategories first"
  -> if category has products:
     -> Show dialog: "Move {n} products to:" + category select dropdown
     -> User selects target category
     -> Move all products: products.forEach(p => updateProduct({ ...p, categoryId: targetId }))
     -> Update productCount on both categories
  -> dispatch DELETE_CATEGORY
  -> toast.success('Category deleted')
```

---

## Category Form (Add/Edit)

```typescript
const categorySchema = z.object({
  name: z.string().min(2, 'Category name is required').max(50),
  description: z.string().max(200).optional().default(''),
  parentId: z.string().nullable().default(null),
  color: z.string().regex(/^#[0-9a-fA-F]{6}$/, 'Invalid color').default('#3b82f6'),
});
```

Color picker: use a simple preset color palette (8-10 colors) with option for custom hex input.

---

## Sorting Categories

```typescript
type CategorySortOption = 'name_asc' | 'name_desc' | 'productCount_desc' | 'productCount_asc' | 'updatedAt_desc';
```

| Option | Label | Logic |
|--------|-------|-------|
| `name_asc` | Name (A-Z) | `a.name.localeCompare(b.name)` |
| `name_desc` | Name (Z-A) | `b.name.localeCompare(a.name)` |
| `productCount_desc` | Most Products | `b.productCount - a.productCount` |
| `productCount_asc` | Fewest Products | `a.productCount - b.productCount` |
| `updatedAt_desc` | Recently Updated | `b.updatedAt.localeCompare(a.updatedAt)` |

---

## Searching Categories

```typescript
function filterCategories(categories: CategoryTreeNode[], searchTerm: string): CategoryTreeNode[] {
  if (!searchTerm) return categories;
  const lower = searchTerm.toLowerCase();

  return categories
    .map(cat => ({
      ...cat,
      children: cat.children.filter(sub =>
        sub.name.toLowerCase().includes(lower)
      ),
    }))
    .filter(cat =>
      cat.name.toLowerCase().includes(lower) || cat.children.length > 0
    );
}
```

- Search filters by category name
- Matching categories auto-expand to show products
- Also searches product names within categories (if product name matches, parent category is shown expanded)

---

## Responsive Behavior

| Screen Size | Layout |
|---|---|
| Desktop (`lg:`) | Full accordion with product mini-tables |
| Tablet (`md:`) | Same layout, slightly more compact |
| Mobile (default) | Accordion with product cards stacked vertically |

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Category with 0 products | Show "No products" empty state in expanded content |
| Delete category with subcategories | Block deletion, show "Remove subcategories first" |
| Search matches no categories | Show "No categories match your search" empty state |
| Category name very long | Truncate with ellipsis, full name in tooltip |
| Orphaned products (categoryId not found) | Show in an "Uncategorized" virtual group at the bottom |
| Product moved to different category | productCount updates on both source and target categories |
| All categories collapsed | Show helpful text: "Click a category to see its products" |

---

## Accessibility

- Accordion sections openable/closable with Enter or Space
- Arrow keys navigate between category headers
- Screen readers announce "expanded" or "collapsed" state
- Focus moves into expanded content when section opens
- Context menu keyboard accessible via three-dot button
- Subcategory indentation conveyed via `aria-level` attribute

---

## Test Scenarios

| Test | Type |
|------|------|
| Renders all categories with correct product counts | Component |
| Expanding a category shows its products | Component |
| Only one category expanded at a time (single mode) | Component |
| Subcategories render nested inside parent | Component |
| Search filters and auto-expands matching categories | Component |
| Add category creates new category in context | Integration |
| Delete category with product reassignment | Integration |
| Sort changes category order | Component |
| Empty category shows empty state message | Component |
| Category color dot matches category.color | Component |
