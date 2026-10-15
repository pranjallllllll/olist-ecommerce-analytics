# **Project Decision Log**

This file records the important data preparation choices, data-quality issues, and precautions for future analysis. Original files in `data/raw/` remain unchanged.

## **Stage 1: Data Exploration**

### 1. Analysis Period

- The main analysis period is January 2017 to August 2018 because order activity is sparse outside this range.
- All orders are retained, including those outside the selected period.

### 2. Customer Identification

- `customer_id` identifies a customer record associated with an order.
- `customer_unique_id` identifies the actual customer and is used for repeat-purchase analysis.

### 3. Orders Without Items

- Retained 775 orders without matching order items because they are non-delivered orders, mainly canceled or unavailable.

### 4. Table Relationships

- Orders can have multiple items, payment records, and reviews.
- These relationships must be handled carefully when joining tables to avoid duplicate rows and inflated totals.

## **Stage 2: Data Cleaning and Preparation**

### 5. Product Categories

- Created `product_category_english` while preserving the original Portuguese category column.
- Manually translated `pc_gamer` to `gaming_pc` and `portateis_cozinha_e_preparadores_de_alimentos` to `small_kitchen_appliances`.
- Labeled 610 missing English category values as `unknown`.

### 6. Product Measurements

- Changed four zero product weights to `NaN`.
- Kept products with missing measurements instead of deleting records or inventing replacement values.

### 7. Reviews

- Kept the latest review per order, reducing 99,224 records to 98,673.
- Retained and flagged 71 reviews created before the purchase date.
- Removed review comment title and message columns because the current analysis focuses on review scores.

### 8. Low Reviews

- Created `is_low_review`: 1 for scores 1–2 and 0 for scores 3–5.
- Kept the value missing for orders without reviews; 768 orders have no review.

### 9. Orders and Revenue

- Retained all 99,441 orders.
- Used `include_in_revenue` to exclude 625 canceled orders and 609 unavailable orders from eligible revenue analysis.
- Preserved the original order and item records for other analyses and revenue reconciliation.
- Delivery metrics are calculated only for delivered orders with valid timestamps (96,470 of 96,478), because orders that were never delivered have no delivery time.

### 10. Delivery Timestamps

- Preserved original timestamps without inventing missing dates or correcting unusual values.
- Identified 14 delivered orders missing approval timestamps, two missing carrier handoff timestamps, and eight missing customer delivery timestamps.
- Identified 166 carrier handoffs before purchase and 23 deliveries before carrier handoff.
- Invalid or missing timestamp pairs are excluded only from the affected delivery calculations.

### 11. Delivery Features

- Created `delivery_days`, `estimated_days`, `delay_days`, `is_late`, and `delay_bucket`.
- Categorized delays as on time/early, 1–3 days late, 4–7 days late, and 8+ days late.

### 12. Geolocation

- Aggregated 1,000,163 geolocation records into 19,015 ZIP code prefixes using median latitude and longitude.

### 13. Revenue and Geographic Features

- Aggregated item prices and freight by `order_id` before creating the order-level dataset.
- Added `seller_state`, `customer_state`, and `same_state` to `order_items_cleaned` for geographic analysis.

### 14. Analysis Period and Data Types

- Used `in_analysis_period` to flag orders within the selected period; retained the 349 orders outside it.
- Used nullable integer types for `is_late`, `is_low_review`, and `review_score` where applicable, preserving missing values.

### 15. Revenue Validation

- The cleaned order-value total matches the original item-price total of **R$13,591,643.70**.

## **Future Considerations**

- **Seller-customer distance:** `seller_customer_distance_km` is optional and has not been created. The cleaned geolocation data is available for future use.
- **Payment aggregation:** `payments` remains at payment-record level. Aggregate by `order_id` before joining to order-level data because 2,961 orders have multiple payment records.
- **Database validation:** During Stage 3, validate PostgreSQL row counts, keys, and relationships against the cleaned datasets.

## **Mistakes and Pitfalls**

- **Category mapping:** The initial mapping incorrectly labeled 13 products from two valid but untranslated categories as `unknown`, giving 623 instead of 610. Manual translations and validation corrected this.
- **Category preservation:** The original Portuguese category column was initially overwritten. The final version preserves it and creates a separate English column.
- **Revenue validation:** An early revenue check occurred before the relevant merges. Revenue must be validated after aggregation and merging.
- **Join fan-out:** Joining items, payments, or reviews without accounting for their relationships can inflate totals.
- **Customer identification:** Use `customer_unique_id` for repeat-customer analysis, not `customer_id`.
