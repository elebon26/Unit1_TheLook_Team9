# Unit 1 DIVE Summary – The Look Dataset  
**Caleb Brunton**  
**October 2025**

My goal in this project was to use *The Look* dataset to uncover how customer behavior, discounting, and product seasonality impact the company’s long-term growth. Using AI-assisted SQL in Colab, I generated CTE and window function queries to track three main KPIs — average order value (AOV), profit per order, and month-over-month category volatility. This approach allowed me to connect customer-level patterns to higher-level revenue performance.

Initially, I assumed that repeat buyers would spend noticeably more than first-time buyers and that discount levels would explain most differences in profit. However, the data revealed a different story: AOV for repeat and first-time customers was nearly identical (≈ $86). Profitability also stayed constant across discount tiers, indicating that *The Look* doesn’t heavily use price reductions as a driver of sales. After validating the results, I realized my early prompts were referencing a nonexistent “discount” column — a mistake that inflated assumptions about pricing impact. Once corrected by calculating discounts from the difference between `retail_price` and `sale_price`, my conclusions became far more accurate and reliable.

Through visualization, I used interactive Plotly charts to validate monthly revenue trends and detect gaps between purchase events and completed orders. This confirmed that orders provide a more stable view of actual financial performance than raw event logs. A deep dive into **Skirts × Traffic Source** highlighted another insight: desktop and search-driven traffic tends to deliver steadier AOV than other channels, while email and social campaigns create spikes but less consistency. Combined with category-level analysis, I found that apparel items like *Jumpsuits & Rompers* and *Clothing Sets* show the strongest seasonal fluctuations, offering clear opportunities for targeted marketing.

By the end of the DIVE process, I proposed the following recommendations:

1. **Strengthen loyalty programs** – Since AOV is stable across customer types, growth should focus on encouraging purchase frequency through retention campaigns and loyalty points.  
2. **Introduce time-based campaigns** – Leverage seasonal categories such as *Jumpsuits & Rompers* for short-term promotions aligned with fashion cycles.  
3. **Audit marketing channels** – Reinvest in high-AOV channels like search and desktop, while refining social and email outreach to improve conversion stability.

Overall, this project showed how iterative validation can refine early AI-generated results into business-ready insights. The DIVE process helped me connect clean SQL logic with actionable strategies — transforming raw data exploration into a clear growth roadmap for *The Look*.
