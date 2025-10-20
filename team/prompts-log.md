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


### ⚠️ Prompt 2 – GROUP BY Syntax Error  

**Prompt to Gemini**  
> Extend the rolling revenue query to calculate Month-over-Month (MoM) and Year-over-Year (YoY) growth using window functions and multiple CTEs.  

**Error Returned**

BadRequest: 400 Syntax error: Unexpected keyword GROUP at [72:114]


**FIX**  

**Issue**  
The `GROUP BY` appeared inside a nested SELECT that already had aggregated fields.  
BigQuery does not allow grouping again within an already aggregated CTE.  

---

**Working SQL Snippet**  
```sql
WITH daily AS (
  SELECT DATE(o.created_at) AS dt,
         SUM(oi.sale_price) AS net_revenue
  FROM `bigquery-public-data.thelook_ecommerce.order_items` oi
  JOIN `bigquery-public-data.thelook_ecommerce.orders` o USING(order_id)
  WHERE o.status = 'Complete'
  GROUP BY dt
),
rolling90 AS (
  SELECT dt,
         net_revenue,
         SUM(net_revenue) OVER (ORDER BY dt ROWS BETWEEN 89 PRECEDING AND CURRENT ROW) AS rev_rolling_90d
  FROM daily
),
moyoy AS (
  SELECT dt,
         net_revenue,
         rev_rolling_90d,
         SAFE_DIVIDE(rev_rolling_90d - LAG(rev_rolling_90d, 30) OVER (ORDER BY dt), LAG(rev_rolling_90d, 30) OVER (ORDER BY dt)) AS mom_growth,
         SAFE_DIVIDE(rev_rolling_90d - LAG(rev_rolling_90d, 365) OVER (ORDER BY dt), LAG(rev_rolling_90d, 365) OVER (ORDER BY dt)) AS yoy_growth
  FROM rolling90
)
SELECT * FROM moyoy
ORDER BY dt;

Result
✅ Query executed successfully after isolating each aggregation step into its own CTE.
✅ MoM and YoY growth metrics computed correctly.
💡 Learned that separating logic into stages simplifies debugging and improves clarity.

### ✅ Prompt 3 – Investigate Fix (Discount Drivers)  

**Prompt to Gemini**  
> Focus on one product category (‘Outerwear & Coats’) and one customer segment (‘Female’).  
> Use SQL to attribute revenue and AOV drivers by discount band, channel, and region.  

**FIX**  

**Issue**  
The earlier query referenced incorrect columns for discount calculation.  
The correct fields are `p.retail_price` and `oi.sale_price`.  
Computed discount rate as `(p.retail_price - oi.sale_price) / p.retail_price` and joined necessary tables.  

---

**Working SQL Snippet**  
```sql
SELECT 
  p.category,
  oi.discount_band,
  u.marketing_channel,
  SUM(oi.sale_price) AS revenue,
  AVG(SAFE_DIVIDE(p.retail_price - oi.sale_price, p.retail_price)) AS avg_discount_rate
FROM `bigquery-public-data.thelook_ecommerce.order_items` oi
JOIN `bigquery-public-data.thelook_ecommerce.products` p ON oi.product_id = p.id
JOIN `bigquery-public-data.thelook_ecommerce.users` u ON oi.user_id = u.id
JOIN `bigquery-public-data.thelook_ecommerce.orders` o USING(order_id)
WHERE o.status IN ('Complete', 'Shipped')
  AND p.category = 'Outerwear & Coats'
GROUP BY p.category, oi.discount_band, u.marketing_channel
ORDER BY revenue DESC;

Result
✅ Query executed successfully and produced valid revenue metrics by channel and discount band.
✅ Revealed that Search and Organic channels drove most sales even at 0% discount — disproving AI’s earlier claim that discounts increased revenue.
💡 Demonstrated the importance of cross-checking insights with grouped and filtered breakdowns.



