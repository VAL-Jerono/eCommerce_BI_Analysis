Below is your **clean, structured Markdown (MD) version** of the project brief, ready for GitHub, Colab, or report submission.

---

# 📊 End of Semester Group Capstone Project

**E-Commerce Analytics using Olist Dataset**

---

## 1. Project Overview

You are a **Data Analyst** hired by an e-commerce marketplace (similar to Olist) to improve:

* Customer experience
* Delivery performance
* Revenue visibility

Management suspects that:

1. Delivery delays are increasing in some regions.
2. Certain product categories drive poor review scores.
3. Payment method mix affects order completion and revenue.
4. A small group of sellers causes most customer dissatisfaction.

### 🎯 Objective

In groups of **4–5 members**, you must:

* Build a **clean, analysis-ready data model**
* Perform **EDA and analytical reporting**
* Deliver an **interactive Power BI dashboard**
* Present insights clearly to support business decisions.

---

## 2. Dataset Description

**Dataset:** Brazilian E-Commerce Public Dataset by Olist
**Source:** Kaggle
**Link:** [https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

### Key Tables

* `orders`
* `order_items`
* `order_payments`
* `order_reviews`
* `products`
* `sellers`
* `customers`
* `geolocation`

The dataset is **relational**, enabling multi-dimensional analysis of:

* Orders
* Payments
* Delivery performance
* Geography
* Product attributes
* Reviews
* Seller behavior

---

## 3. Required Analytical Questions

You must answer **at least 12 questions**.
All **Core Questions are mandatory**.

Each question must use **EDA techniques**, including:

* Descriptive statistics
* Grouping & aggregation
* Visualizations: line charts, bar charts, histograms, box plots, maps

---

### 🔹 Core Questions (Mandatory)

1. How do order counts and revenue change over time (monthly/quarterly)?
2. What is the average delivery time and how does it vary by region/state/city?
3. Which regions/categories have the highest delivery delays relative to estimated delivery dates?
4. How do review scores vary by delivery delay (on-time vs late)?
5. Which product categories have the lowest average review scores and highest return-risk signals?
6. Which payment methods are most common and how do they relate to order value and order completion status?

---

### 🔹 Additional Questions (Choose at least 4)

7. Which sellers contribute most to late deliveries (Top 10 by delay rate)?
8. Which sellers contribute most to low review scores?
9. Do higher shipping costs correlate with faster or slower delivery?
10. What is the geographic distribution of orders and top revenue regions?
11. What are the top categories by revenue and by satisfaction?
12. What is the relationship between items per order and delivery delay?
13. What is the seasonality of purchases and delivery performance during peak periods?
14. What portion of orders are early/on-time/late and how has this changed over time?

---

## 4. Data Cleaning & Preprocessing Requirements

Minimum tasks to perform:

### 🔹 Data Quality

* Handle missing values (timestamps, reviews, geolocation)
* Remove or justify duplicate keys:

  * `order_id`
  * `customer_id`
  * `seller_id`
  * `product_id`
* Fix data types:

  * Convert timestamps → datetime
* Outlier detection:

  * `price`
  * `freight_value`
  * `delivery_time`

---

### 🔹 Data Modeling

You must clearly define:

* Join keys
* Join types:

  * Inner joins
  * Left joins
* Treatment of multi-item orders for:

  * Revenue
  * Delivery time
  * Review aggregation

All modeling decisions must be **justified**.

---

## 5. Power BI Dashboard Requirements

Your dashboard must contain **at least 3 pages**.
Recommended structure:

---

### 📄 Page 1 — Executive Overview

**Purpose:** High-level business summary.

**Visuals:**

* KPI Cards:

  * Total Orders
  * Total Revenue
  * Avg Delivery Time
  * On-time Rate
  * Avg Review Score
* Monthly trend:

  * Orders
  * Revenue
* Key Insights Narrative Box:

  * 3–5 bullet insights

---

### 📄 Page 2 — Delivery & Logistics

**Focus:** Operational performance.

**Visuals:**

* Delivery time distribution (histogram)
* Late delivery rate by state/city (map + bar)
* Delay trend over time

---

### 📄 Page 3 — Customer Experience

**Focus:** Satisfaction drivers.

**Visuals:**

* Review score distribution
* Review score vs delivery delay
* Worst categories/sellers by review score

  * With minimum review count filter

---

### 📄 Page 4 — Sales & Product Performance

* Top categories by revenue and volume
* Price vs freight contribution
* Items per order vs order value

---

### 📄 Page 5 — Seller Performance

* Seller rankings:

  * Revenue
  * On-time %
  * Avg review
* Quadrant:

  * High revenue & low satisfaction

---

## 6. Key Deliverables

Each group must submit:

1. ✅ Cleaned Dataset
2. ✅ Data Cleaning & EDA Notebook (Google Colab)
3. ✅ Power BI Dashboard
4. ✅ Oral Presentation

---

## 7. Presentation Guidelines

* 📅 Date: **1st February**
* ⏱️ 7 minutes presentation
* ❓ 3 minutes Q&A
* 👥 All members must participate
* 💻 Demonstrate:

  * Google Colab notebook
  * Power BI dashboard

---

## 8. Evaluation Focus

You will be assessed on:

* Data cleaning quality
* Analytical depth
* Business interpretation
* Dashboard clarity
* Storytelling
* Team participation

---
