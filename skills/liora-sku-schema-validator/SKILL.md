---
name: liora-sku-schema-validator
description: Connects to the Liora ERP & Website Admin Panel to validate product records, detect duplicate SKUs, verify image assets, and enforce the strict manual LKR discount rule.
---
# Liora SKU Schema Validator

Validate product catalog records from the Liora ERP and Website Admin Panel against mandatory schema rules, pricing logic, and duplicate detection thresholds.

## Verification & Execution Procedure

1. Open the Liora ERP Product Catalog table and Website Admin Panel product list.
2. For each product record, verify the following mandatory fields are non-null:
   - `product_name` (Title Case, e.g., "Liora Grand Aurelia Block Heel Sandal - Black")
   - `master_sku` (`LG-FW-[4_DIGIT_CODE]` for Footwear, `LG-HB-[4_DIGIT_CODE]` for Handbags)
   - `variant_skus` (Appended with `-[COLOR_CODE]-[EU_SIZE]`, e.g., `LG-FW-1042-BLK-38`)
   - `category` (`Footwear` or `Handbags`)
   - `unit_cost_lkr` (Numeric > 0)
   - `retail_price_lkr` (Numeric > `unit_cost_lkr`)
   - `manual_discount_lkr` (Numeric >= 0; verify `discount_type` is strictly `FIXED_AMOUNT`, never `PERCENTAGE`)
   - `final_selling_price_lkr` (Must equal `retail_price_lkr - manual_discount_lkr`)
   - `image_urls` (Minimum 3 resolving HTTPS URLs returning HTTP 200)
3. Check for duplicate `master_sku`, `variant_skus`, or fuzzy-matched `product_name` strings (>90% similarity).
4. Output a structured Markdown Exception Table listing any failing SKUs, exact missing/corrupted fields, and the prepared correction payload waiting for Owner approval.

## Exception Table Template

| Master SKU | Variant SKU | Failing Field(s) | Issue Description | Prepared Correction Payload | Approval Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `LG-FW-1042` | `LG-FW-1042-BLK-38` | `discount_type` | Set to `PERCENTAGE` instead of `FIXED_AMOUNT` | `{"discount_type": "FIXED_AMOUNT", "manual_discount_lkr": 1500}` | Pending Owner Approval |
---
name: liora-size-chart-standardizer
description: Maps footwear styles (Heels, Flats, Sandals, Loafers, Sneakers) to exact EU 35–42 foot length measurements in centimeters (cm) and UK equivalents.
---
# Verification & Execution Procedure
1. Read the footwear sub-category and sole mould type from the draft product record.
2. Attach the standardized LIORA GRAND EU-to-CM mapping table:
   - EU 36 = 23.0 cm | EU 37 = 23.5 cm | EU 38 = 24.0 cm | EU 39 = 24.5 cm | EU 40 = 25.0 cm | EU 41 = 25.5 cm | EU 42 = 26.0 cm
3. If the product note specifies "Narrow Toe Box / Pointed Toe", automatically append the fit advisory note: *"Fit Note: For wider feet, we recommend selecting one EU size up."*
4. Save the formatted size table into the draft product metadata.
