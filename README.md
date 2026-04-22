# Retail Store Movie Business Analysis

- **Industry:** Rental & Subscription, This project simulates a **Netflix 2005 DVD-era** rental business

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8759eac-a1d2-4239-afee-1dc7f47237b2" />

---

### 📌 Project Overview

- The analysis focuses on optimizing **Demand, Inventory, and Customer Intelligence** via **Product and Store-level insights**, directly influence **revenue, utilization, and operational efficiency**.

### ⚡ Executive Summary
- Demand is widely distributed across categories, indicating no strong category dominance, with the top category contributing ~7% and even the top 5 categories accounting for just ~35% of total demand.

---

### 🚩 Business Problem

### Business Performance

- Are we growing or declining?
- Are customers increasing?
- Is revenue efficient per transaction?

### What Customers Actually Want (Demand)

- No clarity on which genres/content drive demand
- Risk of investing in the wrong content

### 📦 Inventory Misallocation (Overstock vs Understock)

- Some movies sit idle
- Some are always unavailable
- Capital is wasted in inventory

### 👥 Who Actually Drives Revenue?

- All customers treated equally
- No targeting or retention strategy

### 🏬 Store-Level Inefficiency (Execution Gap)

- Some stores underperform
- Inventory not aligned with local demand


### 📈 No Time-Based Strategy

- Business doesn’t understand seasonality or trends



---

### 🎯 Objective

Build a scalable analytics framework to:

- Track Business Health KPIs
- Identify High-Value Categories 
- Optimize Inventory Allocation
- Understand Customer Targeting
- Provide Store wise seasonality


---

### 📊 Primary KPI Framework ⭐

| Category         | KPIs                                   | Insight                                |
|------------------|-----------------------------|---------------------------------------------------|
| Business Health   |   Total Revenue, Total Rentals, AOV, Active customers  |  Overall financial scale.        |
| Demand            |   Rentals by Category, Top Rating                      |  What do customers want          |
| Inventory	        |   Inventory Velocity, Revenue per Inventory	         |  Are we stocking efficiently?    |
| Customer	        |   CLV, Segmentation, Spending	                         |  Who drives revenue?             |
| Store	            |   Revenue % , Rentals %, Efficiency                    |  Where is execution failing?     |
| Operations	    |   Rental Duration, Late Returns	                     |  Where do we lose efficiency?    |


---


### 🗄️ Dataset

Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Scale:
- **32,000+** rental & payment records
- Across 16+ relational tables
- Entities: Customers, Films, Inventory, Stores, Rentals, payments


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

✅ Insight
> Establishes a single source of truth
> Enables executive decision-making
> Detects growth vs stagnation early

```
### Demand Layer

> Demand Analysis (Category + Rating)

``` sql

-- Category Demand

SELECT 
    c.name AS category,
    COUNT(r.rental_id) AS demand,
    ROUND(
        100 * COUNT(r.rental_id) / SUM(COUNT(r.rental_id)) OVER (), 
        2
    ) AS demand_pct
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film_category fc ON i.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
GROUP BY c.name
ORDER BY demand DESC;


-- Rating Demand

SELECT 
    f.rating,
    COUNT(r.rental_id) AS demand,
    ROUND(
        100 * COUNT(r.rental_id) / SUM(COUNT(r.rental_id)) OVER (), 
        2
    ) AS demand_pct
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
GROUP BY f.rating
ORDER BY demand DESC;

✅ Insight
> Demand is spread out (no dominance)
> Revenue is concentrated (few genres drive money)


```


### Inventory Efficiency Layer

> Demand per Inventory KPI

``` sql

-- 1. Demand per Inventory

SELECT 
    f.title,
    COUNT(r.rental_id) AS demand,
    COUNT(DISTINCT i.inventory_id) AS inventory,
    ROUND(
        COUNT(r.rental_id) / COUNT(DISTINCT i.inventory_id), 
        2
    ) AS demand_per_inventory
FROM film f
JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY f.title
-- ORDER BY demand_per_inventory DESC
ORDER BY demand_per_inventory ASC
LIMIT 10;


✅ Insight
> Top films → ~5 rentals per copy
> Bottom films → ~2 rentals per copy
> Indicates understocking of high-demand content and overstocking of low-demand content

```



### Customer Intelligence Layer 

> Customer Segmentation + CLV Analysis + % 

``` SQL

-- 1. Customer Activity

SELECT 
    customer_id,
    COUNT(rental_id) AS total_rentals
FROM rental
GROUP BY customer_id
ORDER BY total_rentals DESC;


-- 2. Customer Segmentation

SELECT 
    CASE 
        WHEN rental_count >= 30 THEN 'High Value'
        WHEN rental_count BETWEEN 15 AND 29 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment,
    COUNT(*) AS customer_count
FROM (
    SELECT 
        customer_id,
        COUNT(rental_id) AS rental_count
    FROM rental
    GROUP BY customer_id
) t
GROUP BY customer_segment;



-- 3. Revenue Contribution per each customer 


SELECT 
    c.customer_id,
    SUM(p.amount) AS total_spent
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY c.customer_id
ORDER BY total_spent DESC;


✅ Insight
> Strong power-user base
> High retention in mid-tier customers
> Stable top spenders → predictable revenue



```


### STORE LAYER

> Store Revenue + Efficiency Metrics

``` sql

-- 1. Store wise revenue and rentals

SELECT 
    s.store_id,
    COUNT(r.rental_id) AS total_rentals,
    SUM(p.amount) AS revenue
FROM store s
JOIN staff st ON s.store_id = st.store_id
JOIN rental r ON st.staff_id = r.staff_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY s.store_id;


-- 2. Store Efficiency (Inventory vs Demand)

SELECT 
    s.store_id,
    COUNT(r.rental_id) AS demand,
    COUNT(DISTINCT i.inventory_id) AS inventory,
    ROUND(
        COUNT(r.rental_id) / COUNT(DISTINCT i.inventory_id), 
        2
    ) AS demand_per_inventory
FROM store s
JOIN inventory i ON s.store_id = i.store_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY s.store_id;

✅ Insight
> Store with more inventory ≠ higher revenue
> Demand allocation is inefficient




```


### Time & Operations

> Monthly Demand & Revenue Trends

``` sql 

-- 1. How does demand/revenue change over time?

-- Demand Trend
SELECT 
    DATE_FORMAT(r.rental_date, '%Y-%m') AS month,
    COUNT(*) AS demand
FROM rental r
GROUP BY month
ORDER BY month;


-- 2. rev Trend

SELECT 
    DATE_FORMAT(p.payment_date, '%Y-%m') AS month,
    SUM(p.amount) AS revenue
FROM payment p
GROUP BY month
ORDER BY month;


```



---


### ER Diagram

<img width="799" height="521" alt="Sakila - ERD" src="https://github.com/user-attachments/assets/4ffc3c2f-1090-4a1a-88f1-1b94ca97054d" />



---


### ✨ Power BI Implementation

> DAX Measures

``` Powerbi

1. Average Rental Duration: AVG_Duration = AVERAGE(Rental[Duration])

2. Customer Tiering: 

Loyalty_Tier = SWITCH(TRUE(),
[Rental_Count] >= 40, "Elite VIP",
[Rental_Count] >= 20, "Preferred",
"Occasional")

3. Store Revenue Gap: Revenue_Gap = [Store 2 Revenue] - [Store 1 Revenue]


```

---



### 📊 Key Insights
- 💰 Revenue: **$67,416**
- 👥 Customers: **599**
- 📦 Inventory: **4,581 units**
- 🎬 Films: **~1,000 titles**
- ⏱ Avg Rental Duration: **~5 days**

---


### 💡 Business Recommendations


### 🎬 Product 
- Avoids misleading demand signals
- Helps prioritize: High-revenue genres over just high-demand ones


### ⚙️ Inventory
Rebalance inventory:
- Increase copies of high-performing films
- Reduce slow-moving stock


### 👥 Customer
- 🎯 Targeted marketing
- 💰 Loyalty programs
- 📈 Revenue predictability

### 🏬 Store
- Inventory ≠ performance
- Poor allocation strategy

### Time Based 
- Seasonal stocking strategy
- Marketing campaign timing
- Demand forecasting

### ⏳ Operations
- Late returns reduce availability
- Impacts peak demand fulfillment
  

--- 


```
📁 Repository Structure
Movie-Rental-Inventory-Analytics-SQL
│
├── Data
|   ├── Introduction 
│   └── rental_records
│
│
├── SQL_Queries
│   ├── 01_Business Health
│   ├── 02_Demand Analysis 
│   ├── 03_Inventory Gap
│   ├── 04_Customer_Behaviour
│   └── 05_Store & Operations
|
├── ER_Diagram
│   └── DVD_Store_ERD.png
│
├── Visuals
|   ├── Metrics
│   └── rental_dashboard.png
│
├── Power BI Analysis
│   ├── Modeling
│   └── Dax Measures 
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



