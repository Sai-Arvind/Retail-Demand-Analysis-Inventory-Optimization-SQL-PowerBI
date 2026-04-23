# DvD Rental Store: 360° Root Cause of Customer Churn & Operational Inefficiency Analysis

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8759eac-a1d2-4239-afee-1dc7f47237b2" />

---


## 📌 Project Overview

This project analyzes a DVD rental business **(Netflix 2006-era + Walmart-style retail model)** to identify the root causes of declining performance.

The analysis follows a full analytics lifecycle:
**Descriptive → Diagnostic → Early-stage Predictive Indicators**

It uncovers a system-wide failure loop across:
- Inventory (Supply)
- Operations (Process)
- Customers (Demand)

The goal is to transform raw transactional data into actionable business insights and strategic recommendations.


---


### 🚩 Business Problem

The business is experiencing **system-wide customer churn** driven by:

- High operational friction (late return penalties)
- Inefficient inventory allocation (dead stock)

These issues create a **failure loop**:

Inventory inefficiency → Poor customer experience → High penalties → Customer churn


## 🔍 Key Problems Identified

#### 1. Inventory Problem (Supply Side)
- Large volume of underperforming DVDs
- Capital locked in low-demand titles

👉 Result: Poor asset utilization

---

#### 2. Operational Problem (Process)
- ~55% Late Return Rate
- Rental durations too short for customer behavior

👉 Result: Customers frequently penalized

---

#### 3. Customer Problem (Demand Side)
- High customer inactivity across all segments
- No significant difference in spend between active & inactive users

👉 Result: System-wide churn driven by poor experience (not customer value)


---

## 🎯 Objective

Build a scalable analytics framework to:

- Audit business performance using KPIs
- Diagnose root causes of inefficiency
- Identify customer churn patterns
- Provide data-driven business recommendations


---


## Business Eco-system 

<img width="1400" height="1000" alt="dvd-Code_Generated_Image" src="https://github.com/user-attachments/assets/1cb14764-77eb-4784-a776-f10923f5c213" />




## 📊 Key Metrics & KPI Framework ⭐

To evaluate the health of the DVD rental business, the following Key Performance Indicators (KPIs) were defined across Inventory, Operations, and Customer behavior.

| Category   | KPI                     | Definition                                      | Business Purpose                                  | Problem Signal                          |
|------------|--------------------------|--------------------------------------------------|--------------------------------------------------|------------------------------------------|
| Inventory  | Inventory Turnover       | Total Rentals per Film / Total Copies           | Measures how efficiently inventory is utilized    | Low turnover → Dead stock                |
| Inventory  | Asset ROI                | Total Revenue per Film / Replacement Cost       | Evaluates profitability of each title             | ROI < 1 → Loss-making inventory          |
| Inventory  | Revenue per Title        | Total revenue generated per film                | Identifies high vs low performers                 | Low revenue → Poor demand                |
| Operations | Late Return Rate (LRR)   | Late Returns / Total Rentals                    | Measures customer friction due to policies        | High (>40%) → Policy failure             |
| Operations | Avg Rental Delay         | Avg(Return Date - Allowed Date)                 | Measures severity of delays                       | High delay → Unrealistic duration        |
| Operations | Revenue per Rental       | Avg payment per rental                          | Tracks pricing effectiveness                      | Low → Pricing inefficiency               |
| Customer   | Recency                  | Days since last rental                          | Measures customer activity                        | High → Churn risk                        |
| Customer   | Frequency                | Total rentals per customer                      | Measures engagement level                         | Low → Weak retention                     |
| Customer   | Monetary (CLV Proxy)     | Total spend per customer                        | Identifies high-value customers                   | High value + inactive → Revenue risk     |
| Customer   | Churn Rate               | % of inactive customers (>30 days)              | Tracks customer loss                              | High churn → Growth problem              |


---



## 🔗 Key Insight

Customer segmentation revealed that:

- Majority of users are inactive
- Average spend is nearly identical across segments

👉 This confirms that churn is **system-wide**, not limited to low-value users.

The primary driver is **operational friction (late return penalties)** rather than customer behavior differences.


---


## 🗄️ Dataset

Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Scale:
- **32,000+** rental & payment records
- Across **16+ relational tables**
- Entities: Content Stragegy, Revenue Engine, Store Performance, market Segmentation

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
> DAX Measures : Avg Rental Duration, Customer Segmentation, Store Revenue Gap

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

### 🔥 Critical Finding

- ~55% of rentals are returned late  
👉 Indicates a systemic policy failure

- Customer inactivity is widespread  
👉 Not a segmentation issue, but a system issue


---



<img width="1137" height="613" alt="Inventory Project 2 S S" src="https://github.com/user-attachments/assets/c083cc60-7682-4a46-b8c5-0ad3dd4f9cf9" />


---


## 💡 Prescriptive Analysis (Recommendations)

### 1. Fix Rental Policy (Top Priority)
- Increasing rental duration by 1–2 days could reduce late return rates (~55%) and improve customer satisfaction.
- Reduce penalty-driven friction

---

### 2. Optimize Inventory
- Remove underperforming titles (low ROI)
- Reinvest in high-demand content

---

### 3. Customer Recovery Strategy
- Target inactive users with re-engagement offers
- Focus on improving experience rather than pricing
  

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


### ⚠️ Critical Finding: Late Behavior is System-Wide

Customer segmentation revealed that 100% of customers have experienced late returns.

This indicates that late behavior is not user-specific, but rather a system-wide outcome driven by operational policies.

With a ~55% Late Return Rate and ~73% churn rate, the data suggests that:

- Late returns are the **default behavior**
- **Customers** are **consistently exposed to penalties**
- **Friction** is embedded into the business model

👉 This confirms that churn is not driven by low-value users or isolated behavior,
but by a structural mismatch between rental policy and customer usage patterns.

> The system is extracting maximum value from its best users… and potentially burning them out.


``` sql

-- cohort comparison: Tag customers

WITH late_users AS (
    SELECT DISTINCT customer_id
    FROM rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
    WHERE r.return_date > DATE_ADD(r.rental_date, INTERVAL f.rental_duration DAY)
),

customer_activity AS (
    SELECT 
        c.customer_id,
        MAX(p.payment_date) AS last_activity,
        DATEDIFF((SELECT MAX(payment_date) FROM payment), MAX(p.payment_date)) AS recency
    FROM customer c
    JOIN payment p ON c.customer_id = p.customer_id
    GROUP BY c.customer_id
)

SELECT 
    CASE 
        WHEN l.customer_id IS NOT NULL THEN 'Late Users'
        ELSE 'On-Time Users'
    END AS user_type,
    
    COUNT(*) AS customers,
    
    ROUND(AVG(recency),2) AS avg_recency,
    
    ROUND(AVG(CASE WHEN recency > 30 THEN 1 ELSE 0 END)*100,2) AS churn_rate_pct

FROM customer_activity c
LEFT JOIN late_users l 
    ON c.customer_id = l.customer_id

GROUP BY user_type;


-- ~55% Late Return Rate
-- High churn (~73%)
-- No difference between active vs inactive spend

```


# Prescriptive Statement

### 📈 Revenue Impact Simulation 

Reducing the Late Return Rate from ~55% to ~30% is expected to:

- Recover ~15–20% of churned customers
- Reactivate ~80–90 high-value users
- Generate an estimated $8K–$10K in recovered revenue

👉 This demonstrates that fixing operational friction has a direct and measurable financial impact.

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



