# Product Dialog (Edit Product & Restock Confirm)

## What It Does

Dialogs (modals) are used for viewing product details, editing products, confirming restock actions, and confirming destructive operations. On mobile, large dialogs render as bottom sheets via shadcn `Sheet` component.

> **Trigger sources:** Opened from [ProductTable](ProductTable.md) (row actions), product detail page (`/products/:id`), and [StockAlerts](StockAlerts.md) (restock action on alert cards). Edit and Restock are always Dialog overlays — they do NOT navigate to a new URL.

> **Dependencies:** Writes to `ProductContext` (edit, delete), `AlertContext` (via `useAlertEngine` — restock clears alerts, see [StockAlerts.md](StockAlerts.md)), `TransactionContext` (restock creates transaction). Reads from `CategoryContext` and `SupplierContext` (for select dropdowns in edit form).

---

## Component Architecture

```
features/products/components/
├── ProductDetailDialog.tsx       # Read-only product detail view
├── EditProductDialog.tsx         # Edit form in dialog
├── RestockDialog.tsx             # Restock quantity + note form
└── ...

components/shared/
├── ConfirmDialog.tsx             # Reusable confirmation dialog (delete, archive, etc.)
└── ...
```

---

## Dialog Types

### 1. Product Detail Dialog (Large)

**Trigger:** Click product name in table or product card

```typescript
interface ProductDetailDialogProps {
  productId: string;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}
```

#### Layout

```
<Dialog>
  <DialogContent className="max-w-3xl">
    <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
      {/* Left Column */}
      <div>
        <ProductImageGallery images={product.images} />
        <p>{product.description}</p>
        <TagList tags={product.tags} />
        <TransactionHistory productId={product.id} />
      </div>

      {/* Right Column */}
      <div>
        <DetailRow label="SKU" value={product.sku} />
        <DetailRow label="Barcode" value={product.barcode} />
        <DetailRow label="Category" value={categoryName} />
        <DetailRow label="Supplier" value={supplierName} />
        <DetailRow label="Cost Price" value={formatCurrency(product.costPrice)} />
        <DetailRow label="Selling Price" value={formatCurrency(product.sellingPrice)} />
        <DetailRow label="Profit Margin" value={`${calculateProfitMargin(product.costPrice, product.sellingPrice).toFixed(1)}%`} />
        <StockLevelIndicator product={product} />
        <DetailRow label="Location" value={product.location || '—'} />
        <DetailRow label="Weight" value={product.weight ? `${product.weight} kg` : '—'} />
        <DetailRow label="Created" value={format(product.createdAt, 'MMM d, yyyy')} />
        <DetailRow label="Updated" value={format(product.updatedAt, 'MMM d, yyyy')} />
      </div>
    </div>

    <DialogFooter>
      <Button onClick={openEditDialog}>Edit</Button>
      <Button onClick={openRestockDialog}>Restock</Button>
      <Button variant="outline" onClick={handleArchive}>Archive</Button>
      <Button variant="destructive" onClick={openDeleteConfirm}>Delete</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

---

### 2. Edit Product Dialog (Large)

**Trigger:** "Edit" from product detail or table row menu

```typescript
interface EditProductDialogProps {
  product: Product;
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onSave: (updated: UpdateProductInput) => void;
}
```

#### Editable Fields

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| Product Name | text | Yes | 3-100 chars | |
| Description | textarea | No | max 500 chars | |
| SKU | text | - | - | **Read-only** (displayed but not editable) |
| Category | select | Yes | Existing categories | |
| Supplier | select | Yes | Active suppliers only | |
| Cost Price | number | Yes | > 0 | |
| Selling Price | number | Yes | >= costPrice | |
| Min Stock Level | number | Yes | >= 0 | |
| Max Stock Level | number | No | > minStockLevel | |
| Unit | select | Yes | pieces/kg/liters/meters/boxes | |
| Storage Location | text | No | | |
| Tags | tag input | No | | |
| Weight | number | No | > 0 if provided | |
| Dimensions | number x3 | No | All three or none | |

#### Validation Schema

Same as wizard schemas (Step 1 + Step 2 + partial Step 3), but:
- SKU field is excluded (read-only)
- Quantity is not editable here (use Restock dialog instead)

```typescript
const editProductSchema = z.object({
  name: z.string().min(3).max(100),
  description: z.string().max(500).optional(),
  categoryId: z.string().min(1, 'Please select a category'),
  supplierId: z.string().min(1, 'Please select a supplier'),
  costPrice: z.number().positive(),
  sellingPrice: z.number().positive(),
  minStockLevel: z.number().min(0),
  maxStockLevel: z.number().optional(),
  unit: z.enum(['pieces', 'kg', 'liters', 'meters', 'boxes']),
  location: z.string().optional(),
  tags: z.array(z.string()),
  weight: z.number().positive().nullable(),
  dimensions: z.object({ length: z.number(), width: z.number(), height: z.number() }).nullable(),
}).refine(d => d.sellingPrice >= d.costPrice, {
  message: 'Selling price cannot be less than cost price',
  path: ['sellingPrice'],
});
```

#### Save Flow

```
User clicks "Save Changes"
  -> validate with editProductSchema
  -> if invalid: show inline errors, stop
  -> dispatch UPDATE_PRODUCT with { ...changes, updatedAt: new Date().toISOString() }
  -> re-evaluate alerts (price change might affect margin alerts)
  -> toast.success('Product updated successfully')
  -> close dialog
```

---

### 3. Restock Confirmation Dialog (Medium)

**Trigger:** "Restock" from product detail, table row menu, or alert action

```typescript
interface RestockDialogProps {
  product: Product;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}
```

#### Content

```
<Dialog>
  <DialogContent className="max-w-md">
    <DialogHeader>
      <DialogTitle>Restock {product.name}</DialogTitle>
    </DialogHeader>

    <p>Current stock: {product.quantity} {product.unit}</p>

    <Label htmlFor="restockQty">Restock Quantity</Label>
    <Input id="restockQty" type="number" min={1} value={restockQty} />
    {errors.restockQty && <p className="text-sm text-destructive">{errors.restockQty}</p>}

    <Label htmlFor="restockNote">Note (optional)</Label>
    <Textarea id="restockNote" value={note} placeholder="e.g., Monthly reorder from Supplier X" />

    <p>New stock level: {product.quantity + restockQty} {product.unit}</p>

    {product.quantity + restockQty > product.maxStockLevel && (
      <Alert variant="warning">
        This restock will exceed the maximum stock level ({product.maxStockLevel}).
      </Alert>
    )}

    <DialogFooter>
      <Button variant="outline" onClick={close}>Cancel</Button>
      <Button onClick={handleConfirm}>Confirm Restock</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

#### Confirm Restock Flow

```
User clicks "Confirm Restock"
  -> validate: restockQty must be > 0
  -> newQuantity = product.quantity + restockQty
  -> dispatch UPDATE_PRODUCT { quantity: newQuantity, isInStock: true, updatedAt: now }
  -> create StockTransaction { type: 'restock', quantity: restockQty, previousQuantity: product.quantity, newQuantity, note }
  -> if product was out_of_stock: clear that alert
  -> if product was low_stock and newQuantity > minStockLevel: clear that alert
  -> if newQuantity > maxStockLevel: create overstock alert
  -> toast.success(`Restocked ${product.name}: ${product.quantity} → ${newQuantity}`)
  -> close dialog
```

---

### 4. Confirmation Dialog (Small, Reusable)

**Trigger:** Any destructive or irreversible action

```typescript
interface ConfirmDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  description: string;
  confirmLabel: string;           // e.g., "Delete", "Archive"
  confirmVariant?: 'default' | 'destructive';  // default: 'destructive'
  onConfirm: () => void;
}
```

#### Usage Examples

| Action | Title | Description | Confirm Button |
|--------|-------|-------------|---------------|
| Delete product | "Delete Product" | "This will permanently delete {name}. This action cannot be undone." | "Delete" (red) |
| Delete category | "Delete Category" | "Choose what to do with the {count} products in this category." | "Delete Category" (red) |
| Delete supplier | "Delete Supplier" | "This supplier is linked to {count} products. They will be unlinked." | "Delete Supplier" (red) |
| Archive product | "Archive Product" | "{name} will be hidden from the product list. You can restore it later." | "Archive" |
| Bulk delete | "Delete {n} Products" | "This will permanently delete {n} products. This cannot be undone." | "Delete All" (red) |
| Unsaved changes | "Unsaved Changes" | "You have unsaved changes. What would you like to do?" | "Discard" + "Save" |

---

## Dialog Behavior (All Types)

| Behavior | Implementation |
|----------|---------------|
| Close on Escape | Built into shadcn `Dialog` |
| Close on outside click | Built into shadcn `Dialog` |
| Focus trap | Built into Radix UI Dialog primitive |
| Initial focus | First input field (forms) or close button (detail view) |
| Return focus | Automatically returns to trigger element on close |
| Background scroll lock | Built into shadcn `Dialog` |
| Tab cycling | Focus stays within dialog boundaries |

---

## Mobile Behavior

On screens < 768px, use shadcn `Sheet` instead of `Dialog`:

```typescript
const isMobile = useMediaQuery('(max-width: 768px)');

return isMobile ? (
  <Sheet open={open} onOpenChange={onOpenChange}>
    <SheetContent side="bottom" className="h-[90vh]">
      {content}
    </SheetContent>
  </Sheet>
) : (
  <Dialog open={open} onOpenChange={onOpenChange}>
    <DialogContent>{content}</DialogContent>
  </Dialog>
);
```

Bottom sheets slide up, take ~90% of screen height, and are dismissible by swiping down.

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Edit dialog: selling price set below cost | Inline error, Save button disabled |
| Restock dialog: quantity = 0 | Validation error: "Restock quantity must be at least 1" |
| Restock exceeds maxStockLevel | Warning shown but restock is still allowed |
| Delete product with active alerts | Alerts for that product are also deleted |
| Delete last product in category | Category's `productCount` drops to 0, no cascade delete |
| Open detail dialog for deleted product | Should not happen (row removed), but guard with null check -> close dialog |
| Multiple dialogs open | Not allowed; opening a new dialog closes the previous one |

---

## Accessibility

- All dialogs have `aria-labelledby` pointing to `DialogTitle`
- All dialogs have `aria-describedby` pointing to `DialogDescription`
- Keyboard: Escape closes, Tab cycles within, Enter confirms focused button
- Focus trap prevents interaction with background content
- Screen readers announce dialog opening and title
- Confirmation dialogs: destructive button is NOT auto-focused (to prevent accidental deletion)

---

## Test Scenarios

| Test | Type |
|------|------|
| Product detail dialog shows all product fields | Component |
| Edit dialog pre-fills with current product data | Component |
| Edit dialog validates and shows inline errors | Component |
| Save in edit dialog updates product in context | Integration |
| Restock dialog calculates new quantity correctly | Component |
| Restock confirm creates transaction and clears alerts | Integration |
| Confirmation dialog calls onConfirm when confirmed | Component |
| Dialog closes on Escape key | Component |
| Focus returns to trigger element on close | Component |
| Mobile renders Sheet instead of Dialog | Component |
