# Add Product Wizard

## What It Does

A step-by-step form that walks the user through adding a new product to inventory. The process is broken into 3 steps, with a progress indicator at the top showing current position.

> **Dependencies:** Reads from `CategoryContext` (category list for select), `SupplierContext` (active suppliers for select). Writes to `ProductContext` (add product), `AlertContext` (via `useAlertEngine`), and `TransactionContext` (initial stock transaction). See [StockAlerts.md](StockAlerts.md) for alert creation rules.

> **URL:** `/products/new` — Supports query param `?categoryId=<id>` to pre-fill category select (used when navigating from [CategoryAccordion](CategoryAccordion.md)).

---

## Component Architecture

```
features/wizard/
├── components/
│   ├── WizardShell.tsx          # Layout wrapper: step indicator + form content + nav buttons
│   ├── StepIndicator.tsx        # Visual progress bar (3 steps)
│   ├── Step1ProductInfo.tsx     # Product name, SKU, category, supplier, tags, images
│   ├── Step2Pricing.tsx         # Cost price, selling price, margin display
│   └── Step3StockDetails.tsx    # Quantity, unit, min/max levels, location, dimensions
├── hooks/
│   ├── useWizardForm.ts         # Multi-step form state via React Hook Form
│   └── useWizardNavigation.ts   # Step management (next, back, canProceed)
└── validation.ts                # Zod schemas for each step
```

### Component Hierarchy

```
<AddProductWizard>                 # Route component at /products/new
  <WizardShell>
    <StepIndicator currentStep={step} totalSteps={3} />
    {step === 1 && <Step1ProductInfo form={form} />}
    {step === 2 && <Step2Pricing form={form} />}
    {step === 3 && <Step3StockDetails form={form} />}
    <WizardNavigation onBack={goBack} onNext={goNext} onSubmit={handleSubmit} isLastStep={step === 3} />
  </WizardShell>
</AddProductWizard>
```

---

## Step 1: Product Information

| Field | Type | Required | Constraints | shadcn/ui Component |
|-------|------|----------|-------------|-------------------|
| Product Name | text | Yes | 3-100 characters | `Input` |
| Description | textarea | No | max 500 characters | `Textarea` |
| SKU | text | No | Unique; auto-generated via `SKU-${Date.now()}` if blank | `Input` |
| Barcode | text | No | - | `Input` |
| Category | select | Yes | Must select from existing categories | `Select` |
| Supplier | select | Yes | Must select from active suppliers | `Select` |
| Tags | tag input | No | Type + Enter to add, click X to remove | Custom `TagInput` |
| Product Images | file upload | No | Accept image/*, display previews | Custom `ImageUpload` |

---

## Step 2: Pricing

| Field | Type | Required | Constraints | shadcn/ui Component |
|-------|------|----------|-------------|-------------------|
| Cost Price | number | Yes | > 0 | `Input type="number"` |
| Selling Price | number | Yes | >= Cost Price | `Input type="number"` |
| Profit Margin | display only | - | Auto-calculated: `((selling - cost) / selling) * 100` | `<span>` |

---

## Step 3: Stock Details

| Field | Type | Required | Constraints | shadcn/ui Component |
|-------|------|----------|-------------|-------------------|
| Initial Quantity | number | Yes | >= 0 | `Input type="number"` |
| Unit | select | Yes | 'pieces' \| 'kg' \| 'liters' \| 'meters' \| 'boxes' | `Select` |
| Min Stock Level | number | Yes | >= 0 | `Input type="number"` |
| Max Stock Level | number | No | > minStockLevel if provided | `Input type="number"` |
| Storage Location | text | No | - | `Input` |
| Weight | number | No | > 0 if provided | `Input type="number"` |
| Dimensions (L/W/H) | number x3 | No | All three or none | `Input type="number"` x3 |

---

## Validation Schemas (Zod)

```typescript
// Step 1
z.object({
  name: z.string().min(3, 'Product name must be at least 3 characters').max(100),
  description: z.string().max(500).optional().default(''),
  sku: z.string().optional().default(''),
  barcode: z.string().optional().default(''),
  categoryId: z.string().min(1, 'Please select a category'),
  supplierId: z.string().min(1, 'Please select a supplier'),
  tags: z.array(z.string()).default([]),
  images: z.array(z.string()).default([]),
})

// Step 2
z.object({
  costPrice: z.number().positive('Cost price is required'),
  sellingPrice: z.number().positive('Selling price is required'),
}).refine(d => d.sellingPrice >= d.costPrice, {
  message: 'Selling price cannot be less than cost price',
  path: ['sellingPrice'],
})

// Step 3
z.object({
  quantity: z.number().min(0, 'Quantity cannot be negative'),
  unit: z.enum(['pieces', 'kg', 'liters', 'meters', 'boxes']),
  minStockLevel: z.number().min(0, 'Minimum stock level is required'),
  maxStockLevel: z.number().optional(),
  location: z.string().optional().default(''),
  weight: z.number().positive().nullable().default(null),
  dimensions: z.object({
    length: z.number().positive(),
    width: z.number().positive(),
    height: z.number().positive(),
  }).nullable().default(null),
})
```

---

## Step Indicator

| State | Visual |
|-------|--------|
| Completed | Green checkmark icon, solid connecting line |
| Current | Primary color highlight, filled circle |
| Upcoming | Gray outline, dashed connecting line |

### Props Interface

```typescript
interface StepIndicatorProps {
  currentStep: number;  // 1-indexed
  totalSteps: number;
  stepLabels: string[]; // ['Product Info', 'Pricing', 'Stock Details']
}
```

---

## Navigation Logic

```typescript
function useWizardNavigation(form: UseFormReturn) {
  const [step, setStep] = useState(1);

  const goNext = async () => {
    const schema = [step1Schema, step2Schema, step3Schema][step - 1];
    const isValid = await form.trigger(getFieldsForStep(step));
    if (isValid) setStep(s => Math.min(s + 1, 3));
  };

  const goBack = () => setStep(s => Math.max(s - 1, 1));

  return { step, goNext, goBack, isFirstStep: step === 1, isLastStep: step === 3 };
}
```

- **Back** button: disabled on Step 1
- **Next** button: validates current step fields only, then advances
- **Add Product** button: appears on Step 3, validates all fields, then submits

---

## Submission Flow

```
User clicks "Add Product"
  -> validate all 3 schemas
  -> generate id via crypto.randomUUID()
  -> generate SKU if blank: `SKU-${Date.now()}`
  -> check SKU uniqueness against existing products
  -> build Product object with createdAt/updatedAt = now
  -> dispatch ADD_PRODUCT action (-> reducer -> state -> localStorage sync)
  -> if quantity <= minStockLevel, create StockAlert { type: 'low_stock', severity: 'warning' }
  -> if quantity === 0, create StockAlert { type: 'out_of_stock', severity: 'critical' }
  -> create StockTransaction { type: 'restock', quantity: initialQuantity, previousQuantity: 0, newQuantity: initialQuantity }
  -> toast.success('Product added successfully')
  -> navigate('/products')
```

---

## Validation Messages

| Condition | Message |
|---|---|
| No product name entered | "Product name is required" |
| Name too short | "Product name must be at least 3 characters" |
| Duplicate SKU | "A product with this SKU already exists" |
| No category selected | "Please select a category" |
| No supplier selected | "Please select a supplier" |
| Cost price empty/zero | "Cost price is required" |
| Selling price < cost | "Selling price cannot be less than cost price" |
| Negative quantity | "Quantity cannot be negative" |
| Min stock level empty | "Minimum stock level is required" |

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Browser refresh mid-wizard | Form state is lost. No localStorage persistence for draft. |
| Duplicate SKU | Show inline error on Step 1 SKU field. Check on submit, not on blur (to avoid perf issues). |
| No categories exist | Show "Create a category first" message with link to categories page. Disable Next button. |
| No active suppliers exist | Show "Add a supplier first" message with link. Disable Next button. |
| User navigates away (browser back) | No unsaved-changes prompt for the wizard (not worth the complexity for a creation form). |
| Very long product name | Truncated in UI with ellipsis via `truncate` Tailwind class. Full name in tooltip. |

## Custom Components

### TagInput

```typescript
// components/shared/TagInput.tsx
interface TagInputProps {
  tags: string[];
  onChange: (tags: string[]) => void;
  placeholder?: string;
}
```

- Text input + list of removable badge chips
- Type text, press Enter -> adds tag (trimmed, deduplicated)
- Click X on tag -> removes it
- Accessibility: each tag is a `<Badge>` with a `<button aria-label="Remove tag: {tagName}">` for the X

### ImageUpload

```typescript
// components/shared/ImageUpload.tsx
interface ImageUploadProps {
  images: string[];              // array of data URIs or URLs
  onChange: (images: string[]) => void;
  maxImages?: number;            // default: 5
}
```

- Dropzone area with "Click or drag to upload" text
- Accepts `image/*` files only
- Shows preview thumbnails with remove button
- Converts files to base64 data URIs (kept under 200KB each via canvas resize)
- Accessibility: `<input type="file" accept="image/*" aria-label="Upload product images">`

---

## Query Parameter Pre-fill

```typescript
// Read categoryId from URL to pre-fill Step 1
const [searchParams] = useSearchParams();
const prefilledCategoryId = searchParams.get('categoryId');

// In form defaultValues:
const form = useForm({
  defaultValues: {
    categoryId: prefilledCategoryId || '',
    // ... other defaults
  },
});
```

---

- Full keyboard navigation: Tab through fields, Enter to submit current step
- Escape key: navigates back to `/products`
- Error messages linked to fields via `aria-describedby`
- Focus moves to the first field of each new step
- Step indicator has `aria-current="step"` on the active step
- All form fields have associated `<Label>` elements

---

## Test Scenarios

| Test | Type |
|------|------|
| Renders all Step 1 fields | Component |
| Validates required fields and shows errors | Component |
| Advances to Step 2 only when Step 1 is valid | Component |
| Back button returns to previous step with data preserved | Component |
| Profit margin auto-calculates when prices change | Component |
| Full wizard completion creates product in context | Integration |
| Created product appears in product list | E2E |
| Low stock alert created when initial quantity < min level | Integration |
| Duplicate SKU shows error | Integration |
