# Add Product Wizard

## What It Does

A step-by-step form that walks you through adding a new product to your inventory. The process is broken into 3 simple steps, and a progress bar at the top shows where you are.

---

## Step 1: Product Information

- **Product Name** — Give the product a name (required, 3-100 characters)
- **Description** — Describe the product (optional, up to 500 characters)
- **SKU** — A unique product code for internal tracking (required, auto-generated if left blank)
- **Barcode** — The product's barcode number (optional)
- **Category** — Choose which category this product belongs to (required)
- **Supplier** — Select the supplier for this product (required)
- **Tags** — Add labels to help organize and search (e.g., "fragile", "seasonal")
- **Product Images** — Upload one or more images (optional)

---

## Step 2: Pricing

- **Cost Price** — How much you pay for this product (required)
- **Selling Price** — How much you sell it for (required, must be greater than or equal to cost price)
- **Profit Margin** — Automatically calculated and displayed based on cost and selling price
- **Currency** — Shown based on your settings (default: USD)
- **Tax Category** — Select if the product is taxable, tax-exempt, or has a special rate (optional)

---

## Step 3: Stock Details

- **Initial Quantity** — How many units you currently have (required, minimum 0)
- **Unit of Measurement** — Choose from: Pieces, Kg, Liters, Meters, or Boxes (required)
- **Minimum Stock Level** — The lowest quantity before a low stock alert triggers (required)
- **Maximum Stock Level** — The highest quantity you want to keep (optional)
- **Storage Location** — Where the product is stored, e.g., "Warehouse A, Shelf 3" (optional)
- **Weight** — Product weight (optional)
- **Dimensions** — Length, width, and height (optional)

---

## Step Indicator

- Completed steps show a green checkmark
- The current step is highlighted with the primary color
- Upcoming steps appear as outlines
- Steps are connected with lines (solid for completed, dashed for upcoming)

---

## Navigation

- **Back** button takes you to the previous step (disabled on Step 1)
- **Next** button moves you forward after checking that all required fields are filled in
- **Add Product** button appears on the final step

---

## What Happens When You Add a Product

- A unique product ID and SKU are generated
- The product is saved to your inventory
- Stock level is set to the initial quantity you entered
- If the initial quantity is below the minimum stock level, a low stock alert is created
- You are taken to the product listing page
- A success message appears confirming the product was added

---

## Validation Messages

| What went wrong | Message you'll see |
|---|---|
| No product name entered | "Product name is required" |
| Name too short | "Product name must be at least 3 characters" |
| Duplicate SKU | "A product with this SKU already exists" |
| No category selected | "Please select a category" |
| No supplier selected | "Please select a supplier" |
| Cost price empty | "Cost price is required" |
| Selling price less than cost | "Selling price cannot be less than cost price" |
| Negative quantity | "Quantity cannot be negative" |
| Min stock level empty | "Minimum stock level is required" |

---

## Accessibility

- Works fully with keyboard navigation
- Press Enter to submit the current step
- Press Escape to go back to the product listing
- Error messages are clearly linked to the fields they refer to
- Focus automatically moves to the first field on each new step
