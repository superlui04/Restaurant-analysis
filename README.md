# Restaurant Orders Data

## Situation

I have been hired as a Data Analyst at **Taste of the World Cafe**, which launched a new menu at the start of the year. Customer transaction data for this menu is now available for analysis.

---

## Project Focus

I am tasked with conducting an in-depth analysis of transaction data from the newly launched menu to evaluate product performance and customer behavior. The goal is to identify high- and low-performing items, uncover purchasing patterns, and highlight key customer segments based on menu preferences.

---

## Objectives

**Explore the `menu_items` table to:**

- Count the total number of items
- Identify the highest and lowest priced dishes
- Analyze Italian dishes
- Summarize items by category

**Explore the `order_details` table to:**

- Assess transaction patterns and frequency

**Combine both tables using SQL joins to:**

- Determine menu item popularity
- Identify top customer preferences

---

## Data Dictionary

| Table | Field | Description | Type (MySQL) | Type (BigQuery) |
| --- | --- | --- | --- | --- |
| **menu_items** | menu_item_id (PK) | Unique ID for menu item | INT | INTEGER |
|  | item_name | Menu item name | VARCHAR | STRING |
|  | category | Cuisine/category type | VARCHAR | STRING |
|  | price | Price in USD | DECIMAL | FLOAT |
| **order_details** | order_details_id | Unique ID for an order item | INT | INTEGER |
|  | order_id (PK) | Order ID | INT | INTEGER |
|  | order_date | Order date (MM/DD/YY) | DATE | STRING |
|  | order_time | Order time (HH:MM:SS AM/PM) | DATETIME | STRING |
|  | item_id (FK) | Matches `menu_item_id` in `menu_items` | INT | STRING → INT64 (cast) |

---

## BigQuery Setup & Data Upload

- Activated Google Cloud account and enabled billing (initially free trial).
- Created the dataset `restaurant` in the US location in BigQuery.
- Uploaded the CSV file as table `order_details` using source upload with schema auto-detection and skipping the header row.
- Verified data integrity with preview and test queries.

Example test query:

```sql
SELECT *
FROM `restaurant-467804.order_id.order_details`
LIMIT 10;

```

---

## SQL Queries and Analysis

### Order Details Table Exploration

```sql
sql

-- Preview first 1000 rows from order_details
SELECT *
FROM `restaurant-467804.order_id.order_details`
LIMIT 1000;

-- Check date range of orders
SELECT
    MIN(order_date) AS start_date,
    MAX(order_date) AS end_date
FROM `restaurant-467804.order_id.order_details`;

-- Count total unique orders
SELECT
    COUNT(DISTINCT order_id) AS total_unique_orders
FROM `restaurant-467804.order_id.order_details`;

-- Count unique orders per month
SELECT
    DATE_TRUNC(order_date, MONTH) AS month,
    COUNT(DISTINCT order_id) AS unique_orders
FROM `restaurant-467804.order_id.order_details`
GROUP BY month
ORDER BY month;

-- Find order with highest number of unique items
SELECT
    order_id,
    COUNT(DISTINCT item_id) AS unique_items
FROM `restaurant-467804.order_id.order_details`
GROUP BY order_id
ORDER BY unique_items DESC
LIMIT 1;

-- Count orders with more than 12 unique items
WITH order_item_count AS (
  SELECT
    order_id,
    COUNT(DISTINCT item_id) AS unique_items
  FROM `restaurant-467804.order_id.order_details`
  GROUP BY order_id
)
SELECT COUNT(*) AS orders_with_more_than_12_items
FROM order_item_count
WHERE unique_items > 12;

```

---

### Menu Items Table Exploration

```sql
sql

-- Preview first 1000 rows from menu_items
SELECT *
FROM `restaurant-467804.menu.menu_items`
LIMIT 1000;

-- Count distinct menu items
SELECT
    COUNT(DISTINCT item_name) AS menu_count
FROM `restaurant-467804.menu.menu_items`;

-- Find most expensive menu items
SELECT
    item_name,
    price
FROM `restaurant-467804.menu.menu_items`
ORDER BY price DESC;

-- Find least expensive menu items
SELECT
    item_name,
    price
FROM `restaurant-467804.menu.menu_items`
ORDER BY price ASC;

-- Count Italian dishes
SELECT
    COUNT(DISTINCT item_name) AS italian_count
FROM `restaurant-467804.menu.menu_items`
WHERE category = "Italian";

-- Count dishes per category
SELECT
    category,
    COUNT(DISTINCT item_name) AS items
FROM `restaurant-467804.menu.menu_items`
GROUP BY category;

-- Calculate average price per category
SELECT
    category,
    ROUND(AVG(price), 2) AS avg_price
FROM `restaurant-467804.menu.menu_items`
GROUP BY category;

```

---

### Combined Analysis with JOIN

```sql
sql
-- Join order_details with menu_items to link order items with menu details
SELECT *
FROM `restaurant-467804.order_id.order_details` AS orders
LEFT JOIN `restaurant-467804.menu.menu_items` AS menu
    ON SAFE_CAST(NULLIF(orders.item_id, 'NULL') AS INT64) = menu.menu_item_id;

-- Count orders per menu item with category
SELECT
    menu.item_name,
    menu.category,
    COUNT(orders.order_id) AS num_orders
FROM `restaurant-467804.order_id.order_details` AS orders
LEFT JOIN `restaurant-467804.menu.menu_items` AS menu
    ON SAFE_CAST(NULLIF(orders.item_id, 'NULL') AS INT64) = menu.menu_item_id
GROUP BY menu.item_name, menu.category
ORDER BY num_orders DESC;

-- Find top 5 orders by total spend
SELECT
    orders.order_id,
    SUM(menu.price) AS total_spent
FROM `restaurant-467804.order_id.order_details` AS orders
LEFT JOIN `restaurant-467804.menu.menu_items` AS menu
    ON SAFE_CAST(NULLIF(orders.item_id, 'NULL') AS INT64) = menu.menu_item_id
GROUP BY orders.order_id
ORDER BY total_spent DESC
LIMIT 5;

```

---

## Notes

- Used `SAFE_CAST` and `NULLIF` to handle non-integer and null values in `item_id` during joins.
- Queries focus on deriving insights into menu performance and customer behavior.
- Potential future analysis includes customer segmentation and forecasting.
