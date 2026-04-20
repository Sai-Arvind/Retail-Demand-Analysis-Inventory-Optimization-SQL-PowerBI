# Retail Store Movie Business
- **Industry:** Rental & Subscription Retail
- **Product Demand** and **Inventory Optimization** using Customer & Store Insights, "Which of the Top 3 categories Driving 57% of Revenue".

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8759eac-a1d2-4239-afee-1dc7f47237b2" />

---

### ⚡ Executive Summary

1. This project simulates a Netflix (2005-era DVD Operations) & Rent the Runway.
- Business Health
- Operations Inventory Utilization & store level performance
- supply: customer behaviour analysis 
2. Genres like Sports, Sci-Fi, and Animation account for a massive 57% of revenue, while the Rental Frequency shows a huge bulk of "Regular" and "Elite" users.
3. If you lost the "x" inventory, the "Elite VIP" tier (people renting 40+ times) would likely collapse.


---

### 🚩 Business Problem
Key challenges:

- Inventory Mismatch: Why does the store with more stock generate less revenue?
- Loyalty Gaps: Who are the 'Elite' users driving the most value?
- Revenue Leakage: How many potential rentals are lost due to late returns?
- Genre Concentration: Is the business too reliant on specific niches like Sports and Sci-Fi?

---

### 🎯 Objective

To build a scalable analytics framework that:

- Tracks Rental Health (Total Revenue, AOV, and Active Base).
- Identifies High-Value Tiers (Elite VIP vs. Occasional).
- Optimizes Store Inventory (balancing supply vs. demand).
- Detects Late Return Patterns to improve stock availability.


---

### 📊 Primary KPI Framework ⭐

| Category         | KPIs                                   | Insight                                 |
|----------------|----------------------------------------|------------------------------------------|
| Business Health | Total Revenue, Total Rentals, AOV      | Overall financial scale.                 |
| Customer Active | Customers, CLV, Loyalty Tiers          | User retention and value.                |
| Product         | Revenue by Genre, Inventory Velocity   | Content demand vs. supply.               |
| Operations      | Avg Rental Duration, Late Return Count | Asset turnover and efficiency.           |


---


### 📊 Dataset

Dataset Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Dataset **Size**:
- **50,000+** rental and payment records
- Handling **16+ relational tables** representing customers, inventory, films, and stores


---


### 🗄️ SQL Deep-Dive Analysis


``` sql

-- 1. Revenue & Business Health

-- Total Revenue & Average order value
SELECT 
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_order_value 
FROM payment;

```


``` sql
-- Monthly Revenue Trend
SELECT 
    DATE_TRUNC('month', payment_date) AS month,
    SUM(amount) AS monthly_revenue
FROM payment
GROUP BY 1 ORDER BY 1;


```


``` sql


-- 2. Customer Segmentation (CLV)

-- Identifying the "Whales" (Top 10 Customers)
SELECT 
    customer_id, 
    SUM(amount) AS lifetime_value
FROM payment
GROUP BY 1
ORDER BY lifetime_value DESC
LIMIT 10;

```

``` sql
-- 3. Operational Leakage (Late Returns)

-- Late return = return_date > rental_date + rental_duration
SELECT 
    f.title, 
    COUNT(*) AS late_return_count
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.return_date > r.rental_date + INTERVAL f.rental_duration DAY
GROUP BY 1 ORDER BY 2 DESC;

```

---


### ER Diagram

<img width="799" height="521" alt="Sakila - ERD" src="https://github.com/user-attachments/assets/4ffc3c2f-1090-4a1a-88f1-1b94ca97054d" />



---


### ✨ Power BI Implementation
DAX Measures Utilized:

Average Rental Duration: AVG_Duration = AVERAGE(Rental[Duration])

Customer Tiering: 
```dax
Loyalty_Tier = SWITCH(TRUE(),
[Rental_Count] >= 40, "Elite VIP",
[Rental_Count] >= 20, "Preferred",
"Occasional")

Store Revenue Gap: Revenue_Gap = [Store 2 Revenue] - [Store 1 Revenue]




```

---





### 📈 Business Performance Snapshot
Total Revenue: $67,416.51

Active Customers: 599

Avg Rental Duration: 4.99 Days

Inventory Capacity: 4,581 Units across 1,000 Films

---


### 📊 Key Insights

### 👥 Customer Intelligence
1. High Stability: The top 10 customers have a narrow spend range ($193–$211), indicating a very stable power-user base.

2. Retention Success: The bulk of users are "Regulars" (11–19 rentals), showing high stickiness.

### 🛍️ Product & Genre Performance

1. Niche Dominance: Sports (21%) and Sci-Fi (19%) drive nearly half of all revenue.
2. Underperformers: Travel and Music genres show significantly lower engagement, suggesting inventory should be shifted.

### ⚙️ Operational Inefficiency

1. The Store Paradox: Store 2 has more inventory but fewer customers than Store 1.
2. Late Returns: High late return counts in specific genres (Sports) are likely causing "out-of-stock" issues during peak windows.

### 💡 Business Recommendations
- Inventory Rebalancing: Relocate underperforming inventory from Store 2 to Store 1 to meet higher customer foot traffic.
- Late Return Mitigation: Introduce a "Loyalty Bonus" for early returns to increase Inventory Velocity and film availability.
- Targeted Niche Expansion: Since Sports and Sci-Fi are the "hooks," bundle these with lower-performing genres to increase AOV.
- Top-of-Funnel Focus: Launch a "First 3 Rentals Free" campaign to convert the "Casual" segment into the "Regular" tier.




--- 


```
📁 Repository Structure
Movie-Rental-Inventory-Analytics-SQL
│
├── Data
│   └── rental_records.csv
│
├── ER_Diagram
│   └── Sakila_ERD.png
│
├── SQL_Queries
│   ├── 01_revenue_by_genre.sql
│   ├── 02_customer_lifetime_value.sql
│   ├── 03_rental_frequency.sql
│   ├── 04_late_return_analysis.sql
│   └── 05_revenue_by_store.sql
│
├── Visuals
|   ├── Key Dataset Metrics
│   └── rental_dashboard.png
│
├── Advanced_SQL_Analysis
│   ├── CTEs
│   └── Window_Function
│
├── README.md
└── .gitignore

```
---

### ⚙️ Tech Stack

| Tool      | Purpose ⭐                                               |
|-----------|----------------------------------------------------------|
| Excel     | ETL, Power Query, Data Cleaning, Transformation          |
| SQL       | Deep analysis, Aggregations, calculations & segmentation |
| Power BI  | Data modeling, DAX measures, dashboards, charts & visualization |

---

![movie1](https://github.com/user-attachments/assets/2fb575c5-2957-41b5-a2b9-337ee91fc36d)


### 👤 About Me

**A Sai Arvind**  

📧 Email: saiarvind5081@gmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/saiarvindofficial/  
💻 GitHub: https://github.com/Sai-Arvind  

⭐ If you found this project useful, consider giving it a star.

