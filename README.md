# SQL Data Analytics Project

Contains SQL queries and visualization images.

# intermediate SQL - Sales Analysis

## Overview

Analysis of customer behavior, retention, and lifetime value for an e-commerce company to improve customer retention and maximize revenue.

## Business Questions

Customer Segmentation: Who are our most valuable customers?
Cohort Analysis: How do different customer groups generate revenue?
Retention Analysis: Which customers haven't purchased recently?

## Clean Up Data

🖥️ Query: 0_create_view.sql

```sql
CREATE OR REPLACE VIEW cohort_analysis AS
WITH customer_revenue AS (
    SELECT
        s.customerkey,
        s.orderdate,
        SUM(s.quantity * s.netprice * s.exchangerate) AS total_net_revenue,
        COUNT(s.orderkey) AS num_orders,
        MAX(c.countryfull) AS countryfull,
        MAX(c.age) AS age,
        MAX(c.givenname) AS givenname,
        MAX(c.surname) AS surname
    FROM sales s
    INNER JOIN customer c ON c.customerkey = s.customerkey
    GROUP BY
        s.customerkey,
        s.orderdate
)
SELECT
    customerkey,
    orderdate,
    total_net_revenue,
    num_orders,
    countryfull,
    age,
    CONCAT(TRIM(givenname), ' ', TRIM(surname)) AS cleaned_name,
    MIN(orderdate) OVER (PARTITION BY customerkey) AS first_purchase_date,
    EXTRACT(YEAR FROM MIN(orderdate) OVER (PARTITION BY customerkey)) AS cohort_year
FROM customer_revenue cr;

```

Aggregated sales and customer data into revenue metrics
Calculated first purchase dates for cohort analysis
Created view combining transactions and customer details

## Analysis

1. Customer Segmentation
   🖥️ Query: 1_customer_segmentation.sql

Categorized customers based on total lifetime value (LTV)
Assigned customers to High, Mid, and Low-value segments
Calculated key metrics like total revenue

## 📈 Visualization:

![customer segmentation](/images/6.3_customer_segementation.png)

Customer Segmentation

## 📊 Key Findings:

High-value segment (25% of customers) drives 66% of revenue ($135.4M)
Mid-value segment (50% of customers) generates 32% of revenue ($66.6M)
Low-value segment (25% of customers) accounts for 2% of revenue ($4.3M)

## 💡 Business Insights

High-Value (66% revenue): Offer premium membership program to 12,372 VIP customers, as losing one customer significantly impacts revenue
Mid-Value (32% revenue): Create upgrade paths through personalized promotions, with potential $66.6M → $135.4M revenue opportunity
Low-Value (2% revenue): Design re-engagement campaigns and price-sensitive promotions to increase purchase frequency

2. Customer Revenue by Cohort

## 🖥️ Query: 2_cohort_analysis.sql

```sql
SELECT
    cohort_year,
    SUM(total_net_revenue) AS total_revenue,
    COUNT(DISTINCT customerkey) AS total_customers,
    SUM(total_net_revenue) / COUNT(DISTINCT customerkey) AS customer_revenue
FROM cohort_analysis
GROUP BY
    cohort_year;

-- Title: Customer Revenue by Cohort (Adjusted for time in market)
WITH purchase_days AS (
    SELECT
        customerkey,
        total_net_revenue,
        orderdate - MIN(orderdate) OVER (PARTITION BY customerkey) AS days_since_first_purchase
    FROM cohort_analysis
)

SELECT
    days_since_first_purchase,
    SUM(total_net_revenue) as total_revenue,
    SUM(total_net_revenue) / (SELECT SUM(total_net_revenue) FROM cohort_analysis) * 100 as percentage_of_total_revenue,
    SUM(SUM(total_net_revenue) / (SELECT SUM(total_net_revenue) FROM cohort_analysis) * 100) OVER (ORDER BY days_since_first_purchase) as cumulative_percentage_of_total_revenue
FROM purchase_days
GROUP BY days_since_first_purchase
ORDER BY days_since_first_purchase;

-- Title: Customer Revenue by Cohort (Adjusted for time in market) - Only First Purchase Date
SELECT
    cohort_year,
    SUM(total_net_revenue) AS total_revenue,
    COUNT(DISTINCT customerkey) AS total_customers,
    SUM(total_net_revenue) / COUNT(DISTINCT customerkey) AS customer_revenue
FROM cohort_analysis
WHERE orderdate = first_purchase_date
GROUP BY
    cohort_year;
```

Tracked revenue and customer count per cohorts
Cohorts were grouped by year of first purchase
Analyzed customer revenue at a cohort level

## 📈 Visualization:

![Cohort Analysis](/images/5.2_customer_revenue_normalized.png)

⚠️ Note: This only includes 2 charts.

Customer Revenue by Cohort (Adjusted for time in market) - First Purchase Date

Customer Revenue Normalized

## 📊 Key Findings:

Customer revenue is declining, older cohorts (2016-2018) spent ~$2,800+, while 2024 cohort spending dropped to ~$1,970.
Revenue and customers peaked in 2022-2023, but both are now trending downward in 2024.
High volatility in revenue and customer count, with sharp drops in 2020 and 2024, signaling retention challenges.
💡 Business Insights:

Boost retention & re-engagement by targeting recent cohorts (2022-2024) with personalized offers to prevent churn.
Stabilize revenue fluctuations and introduce loyalty programs or subscriptions to ensure consistent spending.
Investigate cohort differences by applying successful strategies from high-spending cohorts (2016-2018) to newer ones.

## 3. Customer Retention

🖥️ Query: 3_retention_analysis.sql

Identified customers at risk of churning
Analyzed last purchase patterns
Calculated customer-specific metrics
📈 Visualization:

Customer Churn by Cohort Year

## 📊 Key Findings:

Cohort churn stabilizes at ~90% after 2-3 years, indicating a predictable long-term retention pattern.
Retention rates are consistently low (8-10%) across all cohorts, suggesting retention issues are systemic rather than specific to certain years.
Newer cohorts (2022-2023) show similar churn trajectories, signaling that without intervention, future cohorts will follow the same pattern.

## 💡 Business Insights:

Strengthen early engagement strategies to target the first 1-2 years with onboarding incentives, loyalty rewards, and personalized offers to improve long-term retention.
Re-engage high-value churned customers by focusing on targeted win-back campaigns rather than broad retention efforts, as reactivating valuable users may yield higher ROI.
Predict & preempt churn risk and use customer-specific warning indicators to proactively intervene with at-risk users before they lapse.

## Strategic Recommendations

Customer Value Optimization (Customer Segmentation)

Launch VIP program for 12,372 high-value customers (66% revenue)
Create personalized upgrade paths for mid-value segment ($66.6M → $135.4M opportunity)
Design price-sensitive promotions for low-value segment to increase purchase frequency
Cohort Performance Strategy (Customer Revenue by Cohort)

Target 2022-2024 cohorts with personalized re-engagement offers
Implement loyalty/subscription programs to stabilize revenue fluctuations
Apply successful strategies from high-spending 2016-2018 cohorts to newer customers
Retention & Churn Prevention (Customer Retention)

Strengthen first 1-2 year engagement with onboarding incentives and loyalty rewards
Focus on targeted win-back campaigns for high-value churned customers
Implement proactive intervention system for at-risk customers before they lapse

## Technical Details

Database: PostgreSQL
Analysis Tools: PostgreSQL, Dbeaver
Visualization: ChatGPT
