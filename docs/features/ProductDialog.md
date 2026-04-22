# Product Dialog (Edit Product & Restock Confirm)

## What It Does

Dialogs (popups) are used for editing product details, confirming restock actions, and confirming destructive operations like deleting. On mobile, these slide up from the bottom of the screen for easier use.

---

## Types of Dialogs

### 1. Product Detail Dialog (Large)
### 2. Edit Product Dialog (Large)
### 3. Restock Confirmation Dialog (Medium)
### 4. Delete/Action Confirmation Dialog (Small)

---

## 1. Product Detail Dialog

**Opens when:** You click on a product name in the table or product card

### What it shows

**Left side:**
- Product image(s)
- Product description
- Tags (e.g., "fragile", "seasonal", "bestseller")
- Stock transaction history (recent restocks, sales, adjustments)

**Right side:**
- SKU and barcode
- Category
- Supplier name
- Cost price and selling price
- Profit margin (percentage)
- Current stock quantity with level indicator
- Minimum and maximum stock levels
- Storage location
- Weight and dimensions (if set)
- Featured status
- Created and last updated dates

**Bottom:**
- Edit button
- Restock button
- Archive button
- Delete button

---

## 2. Edit Product Dialog

**Opens when:** You click "Edit" from the product detail or the table row menu

### Fields available

- **Product Name** — Required
- **Description** — Optional
- **SKU** — Read-only (cannot be changed after creation)
- **Category** — Choose from existing categories
- **Supplier** — Choose from active suppliers
- **Cost Price** — Required
- **Selling Price** — Required
- **Minimum Stock Level** — Required
- **Maximum Stock Level** — Optional
- **Unit** — Pieces, Kg, Liters, Meters, or Boxes
- **Storage Location** — Optional
- **Tags** — Type and press Enter to add
- **Weight & Dimensions** — Optional

### Buttons
- **Cancel** — Close without saving
- **Save Changes** — Save and close

---

## 3. Restock Confirmation Dialog

**Opens when:** You click "Restock" from the product detail or table row menu

### What it shows

- Product name and current stock quantity
- **Restock Quantity** — Enter how many units to add (required, must be greater than 0)
- **Note** — Optional note about this restock (e.g., "Monthly reorder from Supplier X")
- Updated stock level preview (current + restock amount)
- Warning if the restock would exceed maximum stock level

### Buttons
- **Cancel** — Close without restocking
- **Confirm Restock** — Add the stock and save

### What happens after confirming
- Product quantity is updated
- A stock transaction record is created
- If the product was out of stock, its status changes to "In Stock"
- If the product was in the low stock alerts, the alert is cleared
- A success message appears confirming the restock

---

## 4. Confirmation Dialogs

**Opens when:** You are about to do something that cannot be undone

### Examples

| Action | Dialog Title | Confirm Button |
|---|---|---|
| Delete a product | "Delete Product" | "Delete" (red) |
| Delete a category | "Delete Category" | "Delete Category" (red) |
| Delete a supplier | "Delete Supplier" | "Delete Supplier" (red) |
| Archive a product | "Archive Product" | "Archive" |
| Bulk delete products | "Delete N Products" | "Delete All" (red) |
| Leave with unsaved changes | "Unsaved Changes" | "Discard" or "Save" |

Each confirmation dialog includes a message explaining what will happen, plus Cancel and Confirm buttons.

---

## Dialog Behavior

- **Press Escape** — Closes the dialog
- **Click outside the dialog** — Closes it
- **Tab key** — Cycles through elements inside the dialog only (won't go to the background)
- **Focus** — Automatically placed on the first field (for forms) or close button (for detail view)
- **After closing** — Focus returns to whatever you clicked to open the dialog
- Background scrolling is disabled while a dialog is open

---

## Mobile Behavior

On small screens (phones), dialogs slide up from the bottom instead of appearing centered, taking up most of the screen height. This makes them easier to reach and interact with on a touchscreen.

---

## Accessibility

- All dialogs are properly labeled for screen readers
- Keyboard navigation works fully within the dialog
- Escape key always closes the dialog
- Focus is trapped inside the dialog while it's open
- Focus returns to the trigger element when the dialog closes
