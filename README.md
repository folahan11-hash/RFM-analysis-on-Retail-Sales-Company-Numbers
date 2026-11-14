# RFM-analysis-on-Retail-Sales-Company-Numbers

## Executive summary
To help maximize marketing ROI and reduce customer churn, I performed an **RFM (Recency, Frequency, Monetary) Analysis** on a retail transaction dataset using **PostgreSQL (pgAdmin)**. By scoring customers from 1 to 5 across their purchasing habits, I segmented the customer base into actionable groups—identifying top revenue drivers ("Champions") as well as high-value buyers who are slipping away ("At Risk")

## Key Business Insights and Recommendations
* **Top Revenue Drivers (Champions):** **[X]%** of customers fall into the *Champions* segment, generating **[X]%** of total revenue. 
  * *Action:* Enroll these buyers in a VIP loyalty tier and offer early access to new products rather than profit-cutting discounts.
* **Retention Opportunity (At Risk):** **[X]%** of customers previously spent above average but have not purchased in over **[X]** days.
  * *Action:* Launch an automated win-back email campaign featuring personalized product recommendations and a time-sensitive incentive.
* **Growth Potential (Potential Loyalists):** **[X]%** of recent buyers have high monetary value but low purchase frequency.
  * *Action:* Cross-sell complementary items within 14 days of their first purchase to build repeat buying habits.

| Customer Segment | RFM Score Criteria | Customer Count | % of Total Base | Avg. Spend ($) |
| :--- | :--- | :--- | :--- | :--- |
| **Champions** | R: 4–5, F: 4–5, M: 4–5 | 0 | 0 % | $[Insert $] |
| **Loyal Customers** | R: 3–5, F: 3–5, M: 3–5 | [Insert #] | [Insert %] | $[Insert $] |
| **Potential Loyalists** | R: 4–5, F: 1–3, M: 1–3 | [Insert #] | [Insert %] | $[Insert $] |
| **At Risk** | R: 1–2, F: 3–5, M: 3–5 | [Insert #] | [Insert %] | $[Insert $] |
| **Hibernating** | R: 1–2, F: 1–2, M: 1–2 | [Insert #] | [Insert %] | $[Insert $] |

---

## 🛠️ Methodology & SQL Techniques
1. **Data Preparation & Anchoring:** Aggregated raw transaction logs at the customer level. Because the dataset is historical, recency was anchored to the dataset's maximum transaction date (`MAX("Date")`) rather than `CURRENT_DATE` to prevent skewed scores.
2. **Quintile Scoring (`NTILE`):** Used Common Table Expressions (CTEs) and the `NTILE(5)` window function to rank customers from 1 to 5 across Recency, Frequency, and Monetary metrics.
3. **Rule-Based Segmentation (`CASE WHEN`):** Combined individual R, F, and M scores into distinct behavioural personas to guide targeted marketing strategies.

### Sample SQL Logic
```sql
SELECT * FROM public."RETAIL DATA"
ORDER BY "Transaction_ID" ASC LIMIT 100

SELECT * FROM "RETAIL DATA";

WITH raw_rfm AS (
    SELECT 
        "Customer",
        (SELECT MAX("Date")::date FROM public."RETAIL DATA") - MAX("Date")::date AS recency_days,
        COUNT("Transaction_ID") AS frequency_count,
        SUM("Total_Amount") AS monetary_value
    FROM public."RETAIL DATA"
    GROUP BY "Customer"
),
rfm_scores AS (
    SELECT 
        "Customer",
        NTILE(5) OVER (ORDER BY recency_days DESC) AS r_score,
        NTILE(5) OVER (ORDER BY frequency_count ASC) AS f_score,
        NTILE(5) OVER (ORDER BY monetary_value ASC) AS m_score
    FROM raw_rfm
)
SELECT 
    "Customer",
    r_score,
    f_score,
    m_score,
    CASE 
        WHEN r_score >= 4 AND f_score >= 4 AND m_score >= 4 THEN 'Champions'
        WHEN r_score >= 3 AND f_score >= 3 THEN 'Loyal Customers'
        WHEN r_score <= 2 AND f_score >= 3 THEN 'At Risk'
        WHEN r_score >= 4 AND f_score <= 2 THEN 'Potential Loyalists'
        ELSE 'Hibernating'
    END AS customer_segment
FROM rfm_scores;
```
---

## 📂 Repository Structure
* `sql/rfm_analysis.sql` — Complete, commented PostgreSQL script from raw aggregation to final segmentation.
* `data/` — Sample CSV of the retail dataset / link to data source.

---

