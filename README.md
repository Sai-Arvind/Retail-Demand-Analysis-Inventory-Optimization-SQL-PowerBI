# DvD Rental Store: 360° Root Cause of Customer Churn Analysis


<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8759eac-a1d2-4239-afee-1dc7f47237b2" />

---


## ⚡ Executive Summary
- 55% rentals returned late
- 73% customer churn across all segments
- **Root cause:** rental policy mismatch
- Fixing friction can recover ~$8K–$10K revenue

---


## 📌 Project Overview

This project is simulation of DVD rental business **(Netflix 2006-era & Walmart retail)**.

Analytics lifecycle:
**Descriptive, Diagnostic** and Early stage of **Predictive** Indicators are covered.

---


### 🚩 Business Problem

The business is experiencing **system-wide customer churn** 

| Category   | Problem     |     Question                                                                 |
|------------|-------------|------------------------------------------------------------------------------|
| Inventory  | Supply      |    Is inventory aligned with customer demand?                                  |
| Operations | Process     |    Are rental policies (duration, returns) negatively impacting customers?     |
| Customer   | Demand      |    Is churn driven by low demand or poor customer experience?                  |


---

## 🎯 Objective

Build a scalable analytics framework to:

- Audit business performance
- Diagnose root causes of operational inefficiency
- Identify customer churn patterns
- Provide data-driven business recommendations


---


## Business Eco-system 

<img width="1400" height="1000" alt="dvd-Code_Generated_Image" src="https://github.com/user-attachments/assets/1cb14764-77eb-4784-a776-f10923f5c213" />




## 📊 Key Metrics & KPI Framework ⭐

To evaluate the health of the DVD rental business, the following Key Performance Indicators (KPIs) were defined across Inventory, Operations, and Customer behavior.

| Category   | KPI                     | Measure                                          |           Purpose                                  | 
|------------|--------------------------|-------------------------------------------------|--------------------------------------------------|
| Inventory  | Inventory Turnover       | Total Rentals per Film / Total Copies           | Efficiently inventory is utilized                 | 
| Inventory  | Asset ROI                | Total Revenue per Film / Replacement Cost       | understand profitability of each title             | 
| Inventory  | Revenue per Title        | Total revenue generated per film                | Identifies high vs low performers                 | 
| Operations | Late Return Rate (LRR)   | Late Returns / Total Rentals                    |  customer friction due to policies        | 
| Operations | Avg Rental Delay         | Avg(Return Date - Allowed Date)                 | severity of delays                       | 
| Operations | Revenue per Rental       | Avg payment per rental                          | Tracks pricing effectiveness                      |
| Customer   | Recency                  | Days since last rental                          | customer activity                        | 
| Customer   | Frequency                | Total rentals per customer                      |  engagement level                         | 
| Customer   | Monetary (CLV Proxy)     | Total spend per customer                        | Identifies high-value customers                   | 
| Customer   | Churn Rate               | % of inactive customers (>30 days)              | Tracks customer loss                              | 



---


## 🗄️ Dataset

Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Scale:
- **32,000+** rental & payment records
- Across **16+ relational tables**

### ER Diagram

<img width="799" height="521" alt="Sakila - ERD" src="https://github.com/user-attachments/assets/4ffc3c2f-1090-4a1a-88f1-1b94ca97054d" />

---


## ⚙️ SQL Deep-Dive Analysis
 
``` sql


-------------------------------------------------------- 1. Auditing

> Revenue, Rentals, AOV, Active Customers


-- 1. Revenue
select sum(amount) as revenue
from payment;

-- 2. Rentals
select count(*) as total_rentals
from rental;

-- 3. Active Customers
select count(distinct customer_id) as active_customers
from rental;

-- 4. AOV
select avg(amount) as avg_order_value
from payment;


-------------------------------------------------------- 2. Inventory KPIs

> Inventory Turnover, Asset ROI, Revenue per Title, Demand per Copy

-- 1. Inventory Turnover
SELECT 
    f.title,
    COUNT(r.rental_id) AS total_rentals,
    COUNT(DISTINCT i.inventory_id) AS total_copies,
    ROUND(COUNT(r.rental_id) / COUNT(DISTINCT i.inventory_id), 2) AS inventory_turnover
FROM film f
JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY f.title
ORDER BY inventory_turnover DESC;




-- 2. Asset ROI
SELECT 
    f.title,
    f.replacement_cost,
    SUM(p.amount) AS total_revenue,
    ROUND(SUM(p.amount) / f.replacement_cost, 2) AS asset_roi
FROM film f
JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
LEFT JOIN payment p ON r.rental_id = p.rental_id
GROUP BY f.title, f.replacement_cost
ORDER BY asset_roi DESC;
    
-- 3. Revenue per Title
SELECT 
    f.title,
    SUM(p.amount) AS total_revenue
FROM film f
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY f.title
ORDER BY total_revenue DESC;

-- 4. Demand per Copy
SELECT 
    f.title,
    COUNT(r.rental_id) AS total_rentals,
    COUNT(DISTINCT i.inventory_id) AS copies,
    ROUND(COUNT(r.rental_id) / COUNT(DISTINCT i.inventory_id), 2) AS demand_per_copy
FROM film f
JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY f.title
ORDER BY demand_per_copy DESC;


-------------------------------------------------------- 3. Operational KPIs

> Late Return Rate (LRR), Avg Rental Delay, Revenue per Rental
-- 5. Late Return Rate (LRR)
SELECT 
    COUNT(*) AS total_returns,
    SUM(
        CASE 
            WHEN r.return_date > DATE_ADD(r.rental_date, INTERVAL f.rental_duration DAY)
            THEN 1 ELSE 0 
        END
    ) AS late_returns,
    ROUND(
        SUM(
            CASE 
                WHEN r.return_date > DATE_ADD(r.rental_date, INTERVAL f.rental_duration DAY)
                THEN 1 ELSE 0 
            END
        ) * 100.0 / COUNT(*), 2
    ) AS late_return_rate_pct
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.return_date IS NOT NULL;



-- 6. Avg Rental Delay
SELECT 
    ROUND(AVG(
        DATEDIFF(
            r.return_date,
            DATE_ADD(r.rental_date, INTERVAL f.rental_duration DAY)
        )
    ), 2) AS avg_delay_days
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.return_date IS NOT NULL
AND r.return_date > DATE_ADD(r.rental_date, INTERVAL f.rental_duration DAY);


-- 7. Revenue per Rental
SELECT 
    ROUND(AVG(p.amount), 2) AS avg_revenue_per_rental
FROM payment p;

-- 8. Rental Duration Utilization
SELECT 
    ROUND(AVG(
        DATEDIFF(r.return_date, r.rental_date)
    ), 2) AS avg_actual_days,
    ROUND(AVG(f.rental_duration), 2) AS avg_allowed_days
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.return_date IS NOT NULL;


-------------------------------------------------------- 👥 4. Customer KPIs (RFM + Churn)

> Recency, Frequency,  Monetary (CLV Proxy), Recency Segmentation Query

-- 9. Recency

SELECT 
    c.customer_id,
    SUM(p.amount) AS total_spent,
    DATEDIFF(
        (SELECT MAX(payment_date) FROM payment),
        MAX(p.payment_date)
    ) AS recency_days
FROM customer c
JOIN payment p 
    ON c.customer_id = p.customer_id
GROUP BY c.customer_id
ORDER BY recency_days DESC;

-- SELECT MAX(payment_date) FROM payment;


-- 10. Frequency
SELECT 
    customer_id,
    COUNT(rental_id) AS total_rentals
FROM rental
GROUP BY customer_id
ORDER BY total_rentals DESC;


-- 11. Monetary (CLV Proxy)
SELECT 
    customer_id,
    SUM(amount) AS total_spent
FROM payment
GROUP BY customer_id
ORDER BY total_spent DESC;


-- 12. Churn (Inactive Customers)
SET @max_date = (SELECT MAX(payment_date) FROM payment);

SELECT 
    c.customer_id,
    MAX(p.payment_date) AS last_activity,
    DATEDIFF(@max_date, MAX(p.payment_date)) AS days_inactive
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id
HAVING days_inactive > 30
ORDER BY days_inactive DESC;




-- 13. Recency Segmentation Query 
WITH customer_recency AS (
    SELECT 
        c.customer_id,
        SUM(p.amount) AS total_spent,
        DATEDIFF(
            (SELECT MAX(payment_date) FROM payment),
            MAX(p.payment_date)
        ) AS recency_days
    FROM customer c
    JOIN payment p 
        ON c.customer_id = p.customer_id
    GROUP BY c.customer_id
)

SELECT 
    customer_id,
    total_spent,
    recency_days,
    
    CASE 
        WHEN recency_days <= 160 THEN 'Active'
        WHEN recency_days BETWEEN 161 AND 170 THEN 'At Risk'
        ELSE 'Inactive'
    END AS customer_segment

FROM customer_recency
ORDER BY recency_days DESC;



-- 13. fix cte
SELECT 
    customer_segment,
    COUNT(*) AS customers,
    ROUND(AVG(total_spent), 2) AS avg_spent
FROM (
   WITH customer_recency AS (
    SELECT 
        c.customer_id,
        SUM(p.amount) AS total_spent,
        DATEDIFF(
            (SELECT MAX(payment_date) FROM payment),
            MAX(p.payment_date)
        ) AS recency_days
    FROM customer c
    JOIN payment p 
        ON c.customer_id = p.customer_id
    GROUP BY c.customer_id
)

SELECT 
    customer_id,
    total_spent,
    recency_days,
    
    CASE 
        WHEN recency_days <= 160 THEN 'Active'
        WHEN recency_days BETWEEN 161 AND 170 THEN 'At Risk'
        ELSE 'Inactive'
    END AS customer_segment

FROM customer_recency
) t
GROUP BY customer_segment;


-------------------------------------------------------- 5.Seasonlaity
SELECT 
    DATE_FORMAT(payment_date, '%Y-%m') AS month,
    COUNT(*) AS rentals,
    SUM(amount) AS revenue
FROM payment
GROUP BY month
ORDER BY month;





```


---


## ✨ Power BI Implementation


``` Powerbi

[Inventory Issues] → [Poor Experience] → [Late Returns] → [Penalties] → [Churn]

> DAX Measures:

----------------------------------------------------Avg Rental Duration

Average Rental Duration: AVG_Duration = AVERAGE(Rental[Duration])


-----------------------------------------------------Customer Segmentation

Customer Tiering: 

Loyalty_Tier = SWITCH(TRUE(),
[Rental_Count] >= 40, "Elite VIP",
[Rental_Count] >= 20, "Preferred",
"Occasional")

-----------------------------------------------------Store Revenue Gap

Store Revenue Gap: Revenue_Gap = [Store 2 Revenue] - [Store 1 Revenue]


```

---

## 📊 Key Insights

- 💰 Total Revenue: $67,416  
- 👥 Customers: 599  
- 📦 Inventory: 4,581 units  
- 🎬 Films: ~1,000 titles  
- ⏱ Avg Rental Duration: ~5 days  

---

## 🚨 Critical Finding

- 55% Late Return Rate
- 73% Churn Rate
- 100% customers experienced late returns

*👉 The system is designed such that the average customer fails.*


## 🔄 Root Cause Loop
Inventory → Experience → Late Returns → Penalties → Churn


---



<img width="1137" height="613" alt="Inventory Project 2 S S" src="https://github.com/user-attachments/assets/c083cc60-7682-4a46-b8c5-0ad3dd4f9cf9" />


---


## ⚖️ Trade-off

Penalty Revenue ↓ vs Retention Revenue ↑  
*👉 Retention is long-term growth*


--- 

## 📈 Business Impact

Reducing the Late Return Rate from ~55% to ~30% will:

- Recover ~15–20% of churned customers
- Reactivate ~80–90 high-value users
- Generate an estimated $8K–$10K in recovered revenue

*👉 This demonstrates that fixing operational friction has a direct and measurable financial impact*



---
## 💡 Business Recommendations

### 1. Fix Rental Policy (Top Priority)
- Increasing rental duration by 1–2 days could reduce late return rates (~55%) and improve customer satisfaction.
- Reducing penalty friction will improve customer experience and retention.

---

### 2. Optimize Inventory
- Identify and remove low-ROI titles (Asset ROI < 1).
- Reallocate invest toward high-demand, high-turnover films.

---

### 3. Recovery Risk customers
- Target inactive customers (>30 days) with re-engagement campaigns.
- Prioritize experience improvements over discounts or pricing changes.

--- 

## 📉 If No Changes made in system

- High-value customers will churn first  
- Penalty-driven revenue will decline  
- Inventory ROI will continue to drop  


---


```
## 📁 Repository Structure

dvd-rental-churn-analysis/
│
├── Data/
├── SQL_Queries/
├── ER_Diagram/
├── Visuals/
├── PowerBI/
├── README.md
└── .gitignore

```
---

# Ongoing 

- churn model in Python:

> Logistic Regression
> Target: churn (inactive > X days)
> Cohort analysis customers


---

## ⚙️ Tech Stack

| Tool        | Techniques |
|------------|-----------|
| Excel      | Data Cleaning, Pivot Tables |
| MySQL      | Joins, Aggregations, CTEs |
| Power BI   | Data Modeling, DAX, Dashboards |

---

## 👤 About Me

**Sai Arvind**

📧 Email: saiarvind5081@gmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/saiarvindofficial/  
💻 GitHub: https://github.com/Sai-Arvind    

⭐ If you found this project useful, consider giving it a star.

--- 


![movie1](https://github.com/user-attachments/assets/2fb575c5-2957-41b5-a2b9-337ee91fc36d)



