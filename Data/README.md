# Olist E-Commerce Data Analysis Project

## 📋 Project Overview

This project analyzes the Brazilian E-Commerce dataset from Olist to improve customer experience, delivery performance, and revenue visibility. The analysis addresses critical business concerns including delivery delays, product category performance, payment method impacts, and seller satisfaction metrics.

**Key Business Questions:**
- Are delivery delays increasing in certain regions?
- Which product categories drive poor review scores?
- How does payment method mix affect order completion and revenue?
- Are a small group of sellers causing most customer dissatisfaction?

---

## 📊 Dataset Description

**Source:** [Brazilian E-Commerce Public Dataset by Olist (Kaggle)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

**Dataset Size:** 
- **Customers:** 99,441 records (96,096 unique customers)
- **Orders:** 99,441 records
- **Order Items:** 112,650 records (98,666 unique orders)
- **Products:** 32,951 records (73 categories)
- **Sellers:** 3,095 records
- **Payments:** 103,886 records (99,440 unique orders)
- **Reviews:** 99,224 records
- **Geolocation:** 1,000,163 records (19,015 unique zip codes)
- **Category Translation:** 71 records

### Dataset Structure

```
customers
├── customer_id (PK)
├── customer_unique_id
├── customer_zip_code_prefix
├── customer_city
└── customer_state

orders
├── order_id (PK)
├── customer_id (FK)
├── order_status
├── order_purchase_timestamp
├── order_approved_at
├── order_delivered_carrier_date
├── order_delivered_customer_date
└── order_estimated_delivery_date

order_items
├── order_id (FK)
├── order_item_id
├── product_id (FK)
├── seller_id (FK)
├── shipping_limit_date
├── price
└── freight_value

products
├── product_id (PK)
├── product_category_name
├── product_name_lenght
├── product_description_lenght
├── product_photos_qty
├── product_weight_g
├── product_length_cm
├── product_height_cm
└── product_width_cm

sellers
├── seller_id (PK)
├── seller_zip_code_prefix
├── seller_city
└── seller_state

payments
├── order_id (FK)
├── payment_sequential
├── payment_type
├── payment_installments
└── payment_value

order_reviews
├── review_id (PK)
├── order_id (FK)
├── review_score
├── review_comment_title
├── review_comment_message
├── review_creation_date
└── review_answer_timestamp

geolocation
├── geolocation_zip_code_prefix
├── geolocation_lat
├── geolocation_lng
├── geolocation_city
└── geolocation_state

category_translation
├── product_category_name
└── product_category_name_english
```

---

## 🎯 Analytical Questions

### Core Questions (Mandatory - All 6)

1. **Temporal Revenue Analysis:** How do order counts and revenue change over time (monthly/quarterly)?
2. **Delivery Performance:** What is the average delivery time and how does it vary by region/state/city?
3. **Delivery Delays:** Which regions/categories have the highest delivery delays relative to estimated delivery dates?
4. **Review Score Analysis:** How do review scores vary by delivery delay (on-time vs late)?
5. **Category Risk Assessment:** Which product categories have the lowest average review scores and highest return-risk signals?
6. **Payment Analysis:** Which payment methods are most common and how do they relate to order value and completion status?

### Additional Questions (Minimum 4 Required)

7. **Seller Delay Performance:** Which sellers contribute most to late deliveries (top 10 by delay rate)?
8. **Seller Satisfaction Performance:** Which sellers contribute most to low review scores (top 10 by avg review score)?
9. **Shipping Cost Correlation:** Do higher shipping costs correlate with faster delivery, slower delivery, or no pattern?
10. **Geographic Distribution:** What is the geographic distribution of orders and where are the top revenue regions?
11. **Category Revenue vs Satisfaction:** What are the top categories by revenue, and are they also top by satisfaction?
12. **Order Size Impact:** What is the relationship between number of items per order and delivery delay?
13. **Seasonality Analysis:** What is the seasonality of purchases and does delivery performance degrade during peaks?

---

## 🧹 Data Quality Issues Identified

### Missing Values
- **Orders Table:**
  - `order_approved_at`: 160 missing (0.2%)
  - `order_delivered_carrier_date`: 1,783 missing (1.8%)
  - `order_delivered_customer_date`: 2,965 missing (3.0%)

- **Products Table:**
  - `product_category_name`: 610 missing (1.9%)
  - `product_name_lenght`: 610 missing (1.9%)
  - `product_description_lenght`: 610 missing (1.9%)
  - `product_photos_qty`: 610 missing (1.9%)
  - `product_weight_g`: 2 missing (0.0%)
  - `product_length_cm`: 2 missing (0.0%)
  - `product_height_cm`: 2 missing (0.0%)
  - `product_width_cm`: 2 missing (0.0%)

- **Reviews Table:**
  - `review_comment_title`: 87,656 missing (88.3%)
  - `review_comment_message`: 58,247 missing (58.7%)

### Duplicates
- **Geolocation Table:** 261,831 duplicate rows (26.2%)

### Data Type Issues
- All timestamp columns are currently stored as `object` type (need conversion to `datetime`)

### Key Relationships
- **Customer to Order:** 1:1 (each order has exactly one customer)
- **Order to Order Items:** 1:Many (average 1.13 items per order)
- **Order to Payment:** 1:Many (some orders have multiple payments)
- **Order to Review:** 1:1 (not all orders have reviews - 99,224 reviews vs 99,441 orders)

---

## 🛠️ Data Cleaning & Preprocessing Plan

### 1. Data Type Conversions
```python
# Convert all timestamp columns to datetime
timestamp_columns = [
    'order_purchase_timestamp',
    'order_approved_at',
    'order_delivered_carrier_date',
    'order_delivered_customer_date',
    'order_estimated_delivery_date',
    'review_creation_date',
    'review_answer_timestamp',
    'shipping_limit_date'
]
```

### 2. Missing Value Strategy
- **Order timestamps:** Analyze pattern - may indicate cancelled/pending orders
- **Product attributes:** Consider imputation or separate category for unknowns
- **Review comments:** Missing is valid (users chose not to comment)

### 3. Duplicate Handling
- **Geolocation:** Keep unique zip_code_prefix combinations or aggregate to centroid

### 4. Outlier Detection
- Check `price`, `freight_value`, `payment_value` for anomalies
- Check delivery times for unrealistic values
- Validate geographic coordinates

### 5. Join Strategy
- **Primary join:** `orders` ← `order_items` (LEFT JOIN to keep orders without items)
- **Revenue calculation:** Aggregate at order level for multi-item orders
- **Delivery metrics:** Calculate at order level (latest delivery for multi-item)

### 6. Feature Engineering
```python
# Calculated metrics needed:
- delivery_delay_days = delivered_date - estimated_date
- actual_delivery_time = delivered_date - purchase_date
- is_late = delivery_delay_days > 0
- items_per_order = count(order_items) by order_id
- total_order_value = sum(price + freight_value) by order_id
```

---

## 📈 Power BI Dashboard Structure

### Page 1: Executive Overview
**KPIs:**
- Total Orders
- Total Revenue
- Average Delivery Time
- On-Time Delivery Rate
- Average Review Score

**Visuals:**
- Monthly trend line (orders & revenue)
- Key insights text box (3-5 bullet points)

### Page 2: Delivery & Logistics
**Visuals:**
- Delivery time distribution (histogram)
- Late delivery rate by state (map + bar chart)
- Delivery delay trend over time (line chart)
- Average delivery time by region (table)

### Page 3: Customer Experience
**Visuals:**
- Review score distribution (bar chart)
- Review score vs delivery delay (scatter plot)
- Worst categories by review score (bar chart with min review filter)
- Review score trend over time (line chart)

### Page 4: Sales & Product Performance
**Visuals:**
- Top categories by revenue (bar chart)
- Top categories by order volume (bar chart)
- Price vs freight contribution by category (stacked bar)
- Items per order vs order value (scatter plot)

### Page 5: Seller Performance
**Visuals:**
- Top sellers by revenue (table)
- Seller on-time delivery rate (bar chart)
- Seller average review score (bar chart)
- Revenue vs Satisfaction quadrant (scatter plot)

---

## 📁 Project Deliverables

1. **Cleaned Dataset** (CSV/parquet files)
   - Master dataset with all joins
   - Individual cleaned tables

2. **Jupyter Notebook** (Google Colab)
   - Data inspection
   - Data cleaning code
   - Exploratory Data Analysis (EDA)
   - Statistical summaries
   - Visualizations

3. **Power BI Dashboard** (.pbix file)
   - 5 interactive pages
   - Slicers and filters
   - DAX measures
   - Relationships configured

4. **Presentation** (7 minutes + 3 min Q&A)
   - Business problem overview
   - Key findings
   - Dashboard walkthrough
   - Recommendations

---

## 🚀 Getting Started

### Prerequisites
```bash
# Required Python libraries
pandas
numpy
matplotlib
seaborn
scipy
```

### Installation
```bash
# Clone or download the dataset from Kaggle
# Place CSV files in data/ directory

pip install pandas numpy matplotlib seaborn scipy
```

### Usage
```python
import pandas as pd

# Load datasets
customers = pd.read_csv('data/olist_customers_dataset.csv')
orders = pd.read_csv('data/olist_orders_dataset.csv')
order_items = pd.read_csv('data/olist_order_items_dataset.csv')
products = pd.read_csv('data/olist_products_dataset.csv')
sellers = pd.read_csv('data/olist_sellers_dataset.csv')
payments = pd.read_csv('data/olist_order_payments_dataset.csv')
reviews = pd.read_csv('data/olist_order_reviews_dataset.csv')
geolocation = pd.read_csv('data/olist_geolocation_dataset.csv')
category_translation = pd.read_csv('data/product_category_name_translation.csv')
```

---

## 👥 Team Structure

- **Group Size:** 4-5 members
- **Each member contributes to:**
  - Data cleaning and preprocessing
  - EDA and analysis
  - Dashboard development
  - Presentation preparation

---

## 📅 Timeline

- **Data Exploration:** Week 1
- **Data Cleaning:** Week 2
- **EDA & Analysis:** Week 3-4
- **Dashboard Development:** Week 5-6
- **Presentation Prep:** Week 7
- **Final Presentation:** February 1st

---

## 📊 Expected Insights

Based on initial data inspection, we anticipate findings related to:

1. **Delivery Performance:** Regional variations in delivery times (SP, RJ states dominant)
2. **Payment Preferences:** Credit card dominance in payment methods
3. **Review Patterns:** High percentage of users not leaving comments (88.3% no title)
4. **Geographic Concentration:** Orders concentrated in Southeast Brazil
5. **Multi-item Orders:** Low average (1.13 items/order) suggests single-item purchasing behavior
6. **Seller Distribution:** 3,095 sellers serving 99,441 orders (avg 32 orders per seller)

---

## 📝 Notes

- Dataset timeframe: 2016-2018 (historical data)
- Currency: Brazilian Real (BRL)
- Missing reviews don't indicate poor experience - some users simply don't review
- Geolocation duplicates may represent multiple coordinates for same zip code
- Payment installments common in Brazil (cultural context)

---

## 📞 Support

For questions or issues:
- Review project guidelines document
- Consult with team members
- Reference Kaggle dataset documentation
- Use Power BI tutorial video provided

---

**Project Status:** Data Inspection Complete ✅ | Next: Data Cleaning & EDA

**Last Updated:** January 2026