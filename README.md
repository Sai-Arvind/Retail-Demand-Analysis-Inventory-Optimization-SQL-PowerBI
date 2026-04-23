# DvD Rental Store: Root Cause of Customer Churn & Operational Inefficiency Analytics

- **Industry:** Rental & Subscription
- **Simulation:** **Netflix** (2006 DVD Era) + **Walmart** DVD Rental Model rental business 

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8759eac-a1d2-4239-afee-1dc7f47237b2" />

---

### 📌 Project Overview

> Identifying System-Wide Customer Churn in a DVD Rental Business (Descriptive → Predictive)
> A Data-Driven 360° Root Cause Analysis of Inventory, Operations, and Customer Churn


### ⚡ Executive Summary




---

### 🚩 Business Problem

> Due to Policy & system changes
> The business is experiencing system-wide customer churn driven by high operational friction (late return policies) and inefficient inventory allocation.
> Bad Inventory ➝ Leads to Poor Experience ➝ Causes Late Fees ➝ Creates Friction ➝ Drives Customer Churn


KPI → Business Problem Mapping

| Business Problem                  | Supporting KPIs                                      |
|----------------------------------|------------------------------------------------------|
| Inventory Inefficiency           | Inventory Turnover, Asset ROI, Revenue per Title     |
| Operational Friction (Late Fees) | Late Return Rate, Avg Rental Delay                   |
| Customer Churn                   | Recency, Frequency, Monetary, Churn Rate             |


**1. Inventory Problem (Supply Side)**
- Too many low-performing DVDs
- Capital is stuck → can’t invest in better/new content

👉 Result: Low asset efficiency

**2. Operational Problem (Process Issue)**
- ~55% late return rate
- Rental policy (short duration) is unrealistic

👉 Result: Customers are constantly penalized

**3. Customer Problem (Demand Side)**
- Customers stop renting
- Likely due to repeated bad experiences (late fees)

👉 Result: Revenue loss + declining loyalty



> These KPIs collectively reveal a failure loop where inefficient inventory leads to poor customer experience, high late return rates, and ultimately the churn of customers.



---

### 🎯 Objective

Build a scalable analytics framework to:

- Auditing (Descriptive)
- 


---


### Eco system 

<img width="1400" height="1000" alt="dvd-Code_Generated_Image" src="https://github.com/user-attachments/assets/1cb14764-77eb-4784-a776-f10923f5c213" />





### 📊 Key Metrics & KPI Framework ⭐

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


### 🗄️ Dataset

Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Scale:
- **32,000+** rental & payment records
- Across **16+ relational tables**
- Entities: Content Stragegy, Revenue Engine, Store Performance, market Segmentation

### ER Diagram

<img width="799" height="521" alt="Sakila - ERD" src="https://github.com/user-attachments/assets/4ffc3c2f-1090-4a1a-88f1-1b94ca97054d" />

---


### ⚙️ SQL Deep-Dive Analysis

### Current Business Health 

> Revenue, Rentals, AOV, Active Customers

``` sql

-- Revenue
select sum(amount) as revenue from payment;

-- Rentals
select count(*) as total_rentals from rental;

-- Active Customers
select count(distinct customer_id) as active_customers from rental;

-- AOV
select avg(amount) as avg_order_value from payment;


-------------------------------------------------------- Inventory KPIs
   
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


-------------------------------------------------------- Operational KPIs
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


-------------------------------------------------------- 👥 3. Customer KPIs (RFM + Churn)

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


```





---


# ✨ Power BI Implementation

> DAX Measures


``` Powerbi

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



### 📊 Key Insights
- 💰 Revenue: **$67,416**
- 👥 Customers: **599**
- 📦 Inventory: **4,581 units**
- 🎬 Films: **~1,000 titles**
- ⏱ Avg Rental Duration: **~5 days**

---



<img width="1137" height="613" alt="Inventory Project 2 S S" src="https://github.com/user-attachments/assets/c083cc60-7682-4a46-b8c5-0ad3dd4f9cf9" />


---


### 💡 Prescriptive Analysis (Business Recommendations)


Fix Rental Policy (Top Priority)
> Reduce late return friction
> Adjust rental duration
Optimize Inventory
> Remove dead stock
> Improve availability
Then Customer Actions
> Retarget ALL inactive users
  

--- 


```
📁 Repository Structure
Movie-Rental-Inventory-Analytics-SQL
│
├── Data
|   ├── Introduction 
│   └── 
│
│
├── SQL_Queries
│   ├── 01_
│   ├── 02
│   ├── 03_
│   ├── 04_
│   └── 05_
|
├── ER_Diagram
│   └── DVD_Store_ERD.png
│
├── Visuals
|   ├── 
│
├── Power BI Analysis
│   ├── 
│
├── README.md
└── .gitignore

```
---

### ⚙️ Tech Stack

| Tools      | Techniques ⭐                                               |
|-----------|----------------------------------------------------------|
| Advance Excel  | Pivot Tables, Formulas, Power Query, Cleaning, ETL         |
| MySQL       | Joins, Aggregations, Window Function, Ctes                    |
| Power BI  | Data modeling, Star Schema, DAX measures, dashboards, charts & visualization |

---

### 👤 About Me

**A Sai Arvind**  

📧 Email: saiarvind5081@gmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/saiarvindofficial/  
💻 GitHub: https://github.com/Sai-Arvind  

⭐ If you found this project useful, consider giving it a star.

--- 


![movie1](https://github.com/user-attachments/assets/2fb575c5-2957-41b5-a2b9-337ee91fc36d)



