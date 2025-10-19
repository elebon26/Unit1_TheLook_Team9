# Unit 1 DIVE Summary – The Look Dataset  
**Max Matteucci**  
**October 2025**

The goal of this project was to use *The Look* dataset to understand who drives the company’s growth and how sales patterns have shifted over time. I used SQL CTEs and window functions to calculate monthly growth, repeat purchase behavior, and revenue by category. My first takeaway was that female customers were responsible for most of the company’s revenue, but after validation, I found that the initial interpretation was too simplified.

Once I normalized the data by order count and average order value, I realized that male customers actually spend more per order, while female customers purchase more frequently. The difference was caused by how I joined tables in my early query, which inflated totals at the item level instead of the order level. Fixing that issue completely changed the story and reinforced the importance of checking every assumption before drawing conclusions.

As I moved into visualization, I used interactive Plotly charts to compare categories and regions over time. This made it clear that growth was concentrated in apparel and accessories, while other areas stayed mostly flat. The ability to filter dynamically helped reveal patterns that static charts couldn’t show, and it made the business story much easier to communicate.

By the end of the DIVE process, I proposed two ideas for the company:  
1. Maintain promotional strength in female-driven categories to protect recurring revenue.  
2. Experiment with premium pricing and loyalty offers for male customers who already show higher spending behavior.  

These insights come from both data and validation, not just surface-level trends. Overall, this project taught me how to connect data accuracy with business relevance. The DIVE process helped me slow down, challenge early conclusions, and turn exploration into a story that decision-makers can trust.
