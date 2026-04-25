# DvD Rental Store: 360° Customer Churn Root Cause Analysis

<img width="1024" height="576" alt="image" src="https://github.com/user-attachments/assets/1a5b1185-363f-4dd4-9fea-96e7c9fd5b5d" />


---


## ⚡ Executive Summary
- 55% rentals returned late
- 73% customer churn across all segments
- **Root cause:** rental policy mismatch
- Fixing friction can recover ~$8K–$10K revenue

---


## 📌 Project Overview

This **project is simulation of** DVD rental business **(Netflix 2006-era & Walmart retail)**.

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




## ⭐ KPI Framework

To evaluate Dvd rental business, the following Kpis are defined across Operations, Inventory, and Customer behavior.

### 📊 Auditing 

| KPI                | Measure                           | Purpose |
|--------------------|-----------------------------------|---------|
| Total Revenue      | SUM(payment.amount)               | Overall business performance |
| Total Rentals      | COUNT(rental_id)                 | Tracks business volume |
| AOV                | AVG(payment.amount)              | Average transaction value |
| Active Customers   | COUNT(DISTINCT customer_id)      | Measures customer base size |

### ⚙️ Operations 

| KPI                 | Measure                               | Purpose |
|---------------------|---------------------------------------|---------|
| Late Return Rate    | Late Returns / Total Rentals          | Measures customer friction from policy |
| Avg Rental Delay    | AVG(Return Date - Allowed Date)       | Measures severity of delays |
| Revenue per Rental   | AVG(payment.amount)                   | Tracks pricing effectiveness |

### 📦 Inventory 
| KPI                 | Measure                                     | Purpose |
|---------------------|---------------------------------------------|---------|
| Inventory Turnover  | Total Rentals per Film / Total Copies       | Measures inventory utilization efficiency |
| Asset ROI           | Total Revenue per Film / Replacement Cost   | Evaluates profitability of each title |
| Revenue per Title   | SUM(payment.amount) per film                | Identifies high and low performing titles |


### 👥 Customer

| KPI                | Measure                          | Purpose |
|--------------------|----------------------------------|---------|
| Recency            | Days since last rental           | Measures customer inactivity and churn risk |
| Frequency          | Total rentals per customer       | Measures engagement and usage intensity |
| Monetary (CLV)     | Total spend per customer         | Identifies high-value customers |
| Churn Rate         | % customers inactive > 30 days   | Measures customer loss and retention health |

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


-------------------------------------------------------- 1. Operational


-- 1. Late Return Rate (LRR)
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



-------------------------------------------------------- 2. Inventory

-- 2. Demand per Copy
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


-- 3. Asset ROI
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
    


-------------------------------------------------------- 👥 3. Customer 

-- 4. Recency

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


```

## 📊 Additional Insights

```sql

-------------------------------------------------------- 4. Time trend analysis

-- Month-over-month growth %
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
- late returns affected nearly all active customers.

*👉 The system is designed such that the average customer fails.*

## 🔄 Root Cause Loop
Inventory → Experience → Late Returns → Penalties → Churn

## ⚖️ Trade-off

Penalty Revenue ↓  
*👉 Retention is long-term growth*

---



<img width="1137" height="613" alt="Inventory Project 2 S S" src="https://github.com/user-attachments/assets/c083cc60-7682-4a46-b8c5-0ad3dd4f9cf9" />




---

## 💡 Business Recommendations

### 1. Fix Rental Policy (Top Priority)
- **Increasing rental duration by 1–2 days** could reduce late return rates (~55%) and improve customer satisfaction.
- **Reducing penalty** friction will improve customer experience and retention.

---

### 2. Optimize Inventory
- Identify and **remove low-ROI titles** (Asset ROI < 1).
- **Reallocate invest** toward high-demand, high-turnover films.

---

### 3. Recovery Risk customers
- **Target** inactive customers (>30 days) with re-engagement **campaigns.**
- **Prioritize experience** improvements over discounts or pricing changes.

--- 


## 📈 Business Impact

Reducing the Late Return Rate from ~55% to ~30% will:

- Recover ~15–20% of churned customers
- Reactivate ~80–90 high-value users
- Generate an estimated $8K–$10K in recovered revenue

*👉 This demonstrates that fixing operational friction has a direct and measurable financial impact*




---

### Limitations
- Churn definition assumption (RFM-based)
- No pricing elasticity modeling
- No external market benchmark
- Static simulation dataset (Sakila)
  
---


```
## 📁 Repository Structure

dvd-rental-churn-analysis/
│
├── Data/
├── SQL_Queries/
|  
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



