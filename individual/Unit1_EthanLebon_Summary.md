Unit 1 DIVE Summary – The Look Dataset

Ethan Lebon
October 2025

My goal in this project was to use The Look dataset to identify what truly drives growth for the business, focusing on customer behavior, category performance, and channel effectiveness. I used SQL CTEs and window functions to calculate key metrics such as 90-day revenue trends, repeat purchase behavior, and average order value (AOV). My early findings suggested that discounts were a key revenue driver, but after deeper validation, I discovered that this was a misleading correlation.

After controlling for marketing channels and regions, I found that Search and Organic traffic consistently accounted for most of the company’s revenue—even when no discounts were applied. This meant that revenue growth came from visibility and acquisition, not pricing incentives. Additionally, when testing AI’s insight that “returning customers spend less,” the data technically supported it (AOV ≈ $97 vs $150 for new customers), but validation showed this pattern stemmed from a small returning-customer base, not a true behavioral difference. This highlighted the importance of testing AI-driven insights against data balance and sample size.

For visualization, I built interactive Plotly charts in Colab to track the last 30 days of performance, sales distribution by region and channel, and top-performing categories and products. These visuals made it easier to detect concentration patterns—such as China and the U.S. driving a majority of sales—and communicate insights dynamically.

By the end of the DIVE process, I proposed two key recommendations for the company:

Prioritize channel optimization by investing in Search and Organic visibility rather than broad discounting.

Strengthen retention efforts by creating loyalty programs or targeted incentives to expand the repeat-customer segment.

Together, these insights suggest that The Look should shift focus from discount-based growth to sustainable, channel-driven customer retention. The DIVE process helped refine surface-level assumptions into data-backed strategies that better align with long-term profitability and brand strength.
