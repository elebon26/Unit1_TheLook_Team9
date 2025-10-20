###DEBUGGING PROMPTS:

## 🧩 Debugging Prompts — Investigate Phase (Fail → Fix)

### ❌ Prompt 1 – Initial Attempt
**Prompt to Gemini**
> Write a BigQuery SQL query joining order_items, products, and users to analyze category-level revenue by gender and average discount amount.  
> Include the column i.discount_amount and aggregate results by category and gender.

**Error Returned**

BadRequest: 400 Name discount_amount not found inside i

**FIX**


**Issue**  
The `order_items` table also does not include a `list_price` column.  
The price reference fields exist in the `products` table instead.

---

### ✅ Prompt 2 – Final Fix
**Prompt to Gemini**
 Correct the SQL by referencing the correct price column from the products table.  
 Use p.retail_price instead of list_price to compute the average discount rate as (p.retail_price - i.sale_price)/p.retail_price.  
 Keep CTE structure, joins, and aggregation by category and gender.

**Working SQL Snippet**
```sql
AVG(SAFE_DIVIDE(p.retail_price - i.sale_price, p.retail_price)) AS avg_discount_rate

Result
✅ Query executed successfully.
✅ Joined correct tables (order_items, products, users).
✅ Produced valid metrics for total revenue, discount percent, and average order value by gender and category.
✅ Generated interactive Plotly bar chart grouped by gender.

❌ Prompt 3 - Initial Attempt

Prompt to Gemini

Using the public BigQuery dataset bigquery-public-data.thelook_ecommerce, write SQL to test whether larger discounts increase order volume but reduce profit per order. Compute per-order totals by joining orders and order_items (filter to completed statuses): revenue = SUM(sale_price), discount_amt = SUM(discount), cost_amt = SUM(cost), profit = revenue − cost_amt, and discount_rate = discount_amt / (revenue + discount_amt). Use CTEs and a window function to bucket orders into 5 discount tiers (NTILE over discount_rate). Finally, aggregate by tier to return order count, average revenue, average profit per order, and average discount_rate in ascending tier order.

Error Returned

BadRequest: 400 Name discount not found inside oi at [10:12].

FIX

Issue
The discount column does not exist in the dataset. Derived discount manually from products.retail_price - order_items.sale_price.

✅ Prompt 4 - Final Fix

Prompt to Gemini

Recreate the query by joining orders, order_items, and products. Compute discount as GREATEST(retail_price - sale_price, 0) and include cost and profit calculations. Use NTILE(5) to bucket discount tiers and average results per tier.

Working SQL Snippet

WITH per_order AS (
  SELECT o.order_id,
         SUM(oi.sale_price) AS revenue,
         SUM(GREATEST(p.retail_price - oi.sale_price, 0)) AS discount_amt,
         SUM(p.cost) AS cost_amt,
         SUM(oi.sale_price) - SUM(p.cost) AS profit,
         SAFE_DIVIDE(SUM(GREATEST(p.retail_price - oi.sale_price, 0)),
                     NULLIF(SUM(p.retail_price), 0)) AS discount_rate
  FROM `bigquery-public-data.thelook_ecommerce.orders` o
  JOIN `bigquery-public-data.thelook_ecommerce.order_items` oi ON o.order_id = oi.order_id
  JOIN `bigquery-public-data.thelook_ecommerce.products` p ON oi.product_id = p.id
  WHERE o.status IN ("Complete","Shipped","Delivered","Approved","Closed")
  GROUP BY 1
),
tiered AS (
  SELECT *,
         NTILE(5) OVER(ORDER BY discount_rate) AS discount_tier
  FROM per_order
)
SELECT discount_tier,
       COUNT(*) AS orders,
       AVG(revenue) AS avg_revenue,
       AVG(profit) AS avg_profit_per_order,
       AVG(discount_rate) AS avg_discount_rate
FROM tiered
GROUP BY discount_tier
ORDER BY discount_tier;

❌ Prompt 5 - Initial Attempt

Prompt to Gemini

Using bigquery-public-data.thelook_ecommerce, write SQL to identify which product categories are most seasonal. Build monthly revenue by category (orders × order_items × products; filter completed statuses). Then, with window functions, compute MoM and YoY percent changes per category using LAG. Aggregate to a category-level summary returning avg_abs_mom_change (average absolute MoM change), sd_mom_change, total revenue, and counts of months with MoM/YoY coverage. Order by avg_abs_mom_change (descending) and limit to the top 15 categories.

Error Returned

BadRequest: 400 Name revenue not found inside e at [9:53].

FIX

Issue
The query joined the events table instead of orders, causing missing revenue fields. Replaced events with orders and order_items, and recalculated monthly revenue per category.

✅ Prompt 6 - Final Fix

Prompt to Gemini

Simplify the joins to only orders, order_items, and products. Build monthly revenue per category, use LAG(revenue) to compute MoM % change, and aggregate averages to find top 15 most seasonal product categories.

Working SQL Snippet

WITH monthly AS (
  SELECT DATE_TRUNC(o.created_at, MONTH) AS month,
         p.category,
         SUM(oi.sale_price) AS revenue
  FROM `bigquery-public-data.thelook_ecommerce.orders` o
  JOIN `bigquery-public-data.thelook_ecommerce.order_items` oi ON o.order_id = oi.order_id
  JOIN `bigquery-public-data.thelook_ecommerce.products` p ON oi.product_id = p.id
  WHERE o.status IN ("Complete","Shipped","Delivered","Approved","Closed")
  GROUP BY 1,2
),
calc AS (
  SELECT category,
         month,
         revenue,
         LAG(revenue) OVER(PARTITION BY category ORDER BY month) AS prev_m,
         SAFE_DIVIDE(revenue - LAG(revenue) OVER(PARTITION BY category ORDER BY month),
                     NULLIF(LAG(revenue) OVER(PARTITION BY category ORDER BY month), 0)) AS mom
  FROM monthly
)
SELECT category,
       ROUND(AVG(ABS(mom)), 4) AS avg_abs_mom_change,
       SUM(revenue) AS total_revenue
FROM calc
GROUP BY category
ORDER BY avg_abs_mom_change DESC
LIMIT 15;


