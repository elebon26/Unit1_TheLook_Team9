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


