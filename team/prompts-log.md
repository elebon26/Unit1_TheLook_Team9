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
```
Result
✅ Query executed successfully.
✅ Joined correct tables (order_items, products, users).
✅ Produced valid metrics for total revenue, discount percent, and average order value by gender and category.
✅ Generated interactive Plotly bar chart grouped by gender.



---

Prompt 3

**Final Prompt Used**  
> Using the public BigQuery dataset `bigquery-public-data.thelook_ecommerce`, write a SQL query that compares the average order value (AOV) between first-time and repeat customers. Each user’s first completed order should be labeled as ‘First-time’ and all subsequent orders as ‘Repeat’. Use a CTE to calculate per-order revenue by summing `sale_price` from `order_items`, then apply a window function (`ROW_NUMBER`) to tag the orders. Finally, aggregate by customer type to compute order counts and average AOV.  

**AI Output Evaluation & Refinement**  
- Gemini’s first draft omitted a filter for completed statuses and rounded `AVG()` inside SQL, causing float errors in BigQuery.  
- Fixed by adding `WHERE o.status IN (...)` and moved rounding to Pandas.  
- Final query returned balanced AOVs (≈ $86) for both groups.

---

Prompt 4

**Final Prompt Used**  
> Using the public BigQuery dataset `bigquery-public-data.thelook_ecommerce`, write SQL to test whether larger discounts increase order volume but reduce profit per order. Compute per-order totals by joining `orders` and `order_items` (filter to completed statuses): `revenue = SUM(sale_price)`, `discount_amt = SUM(discount)`, `cost_amt = SUM(cost)`, `profit = revenue − cost_amt`, and `discount_rate = discount_amt / (revenue + discount_amt)`. Use CTEs and a window function to bucket orders into 5 discount tiers (`NTILE` over `discount_rate`). Finally, aggregate by tier to return order count, average revenue, average profit per order, and average discount_rate in ascending tier order.  

**Fail → Fix Example #1**  
- **Fail:** Initial prompt assumed a `discount` column existed in `order_items`, leading to a “Field not found” error and 0 values.  
- **Fix:** Derived discount with `GREATEST(retail_price − sale_price, 0)` from `products`, joined `products` for `cost` and `retail_price`, and re-computed `discount_rate`.  
- **Result:** Query executed successfully and revealed flat profit across discount tiers (≈ \$41–\$48).  

**Additional Refinement**  
- Replaced `NTILE(4)` with `NTILE(5)` for more balanced tiering.  
- Moved aggregation to final CTE to simplify grouping and avoid duplicate rows.  

---

Prompt 5

**Final Prompt Used**  
> Using `bigquery-public-data.thelook_ecommerce`, write SQL to identify which product categories are most seasonal. Build monthly revenue by category (`orders` × `order_items` × `products`; filter completed statuses). Then, with window functions, compute MoM and YoY percent changes per category using `LAG`. Aggregate to a category-level summary returning `avg_abs_mom_change` (average absolute MoM change), `sd_mom_change`, `total_revenue`, and counts of months with MoM/YoY coverage. Order by `avg_abs_mom_change` (descending) and limit to the top 15 categories. Use CTEs throughout.  

**Fail → Fix Example #2**  
- **Fail:** Early prompt joined `events` instead of `orders`, causing duplicate category rows and inconsistent month counts.  
- **Fix:** Removed `events` join, retained only `orders` + `order_items` + `products`, and added `WHERE o.status IN (...)` to filter valid sales.  
- **Result:** Clean output (15 categories) with accurate month-over-month volatility; “Jumpsuits & Rompers,” “Clothing Sets,” and “Skirts” showed highest seasonality.  

**Additional Refinement**  
- Added `STDDEV_POP(mom_pct_change)` as `sd_mom_change` for variability metric.  
- Rounded floats to 4 decimals for Looker compatibility.  

---
