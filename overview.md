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

### Dataset Structure & Relationships

```
customers
├── customer_id (PK)
├── customer_unique_id
├── customer_zip_code_prefix
├── customer_city
└── customer_state

orders
├── order_id (PK)
├── customer_id (FK → customers.customer_id)
├── order_status
├── order_purchase_timestamp
├── order_approved_at
├── order_delivered_carrier_date
├── order_delivered_customer_date
└── order_estimated_delivery_date

order_items
├── order_id (FK → orders.order_id)
├── order_item_id
├── product_id (FK → products.product_id)
├── seller_id (FK → sellers.seller_id)
├── shipping_limit_date
├── price
└── freight_value

products
├── product_id (PK)
├── product_category_name (FK → category_translation)
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
├── order_id (FK → orders.order_id)
├── payment_sequential
├── payment_type
├── payment_installments
└── payment_value

order_reviews
├── review_id (PK)
├── order_id (FK → orders.order_id)
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
├── product_category_name (PK)
└── product_category_name_english
```

### Key Relationships Explained

**1. Customer to Order (1:1)**
- Each order belongs to exactly one customer
- 99,441 customers mapped to 99,441 orders
- However, 96,096 unique `customer_unique_id` suggests ~3,345 repeat customers
- **Analysis implication:** Can identify repeat purchase behavior and customer lifetime value

**2. Order to Order Items (1:Many)**
- Each order can have multiple line items
- 112,650 order items across 98,666 unique orders
- Average: 1.13 items per order
- **Analysis implication:** Most orders are single-item; multi-item orders may have different delivery/satisfaction patterns

**3. Order to Payment (1:Many)**
- Some orders have multiple payment transactions
- 103,886 payment records for 99,440 unique orders
- Average: 1.04 payments per order
- **Analysis implication:** Split payments exist; need aggregation for total order value

**4. Order to Review (1:0..1)**
- Not all orders have reviews
- 99,224 reviews for 99,441 orders (217 orders without reviews)
- 99.8% review rate
- **Analysis implication:** High engagement but missing reviews may correlate with order status

**5. Product to Category (Many:1)**
- 32,951 products across 73 categories
- 610 products (1.9%) have no category assigned
- **Analysis implication:** Uncategorized products need handling strategy

**6. Geographic Relationships**
- Customers, sellers, and geolocation linked via zip code prefix
- 19,015 unique zip codes in geolocation vs 14,994 customer zips
- **Analysis implication:** Can calculate distance between customer and seller

---

## 🎯 Analytical Questions

### Core Questions (Mandatory - All 6)

#### 1. Temporal Revenue Analysis
**Question:** How do order counts and revenue change over time (monthly/quarterly)?

**Analysis Approach:**
- Extract month/quarter from `order_purchase_timestamp`
- Aggregate order counts and sum of payment values
- Calculate growth rates (MoM, QoQ)
- Identify seasonal patterns and trends

**Expected Metrics:**
- Monthly order volume
- Monthly revenue (sum of payment_value)
- Average order value (AOV) by month
- Revenue growth rate
- Order count growth rate

**Visualizations:**
- Dual-axis line chart (orders vs revenue over time)
- Bar chart for quarterly comparisons
- Heatmap showing day-of-week and hour-of-day patterns

---

#### 2. Delivery Performance by Region
**Question:** What is the average delivery time and how does it vary by region/state/city?

**Analysis Approach:**
- Calculate actual delivery time: `order_delivered_customer_date - order_purchase_timestamp`
- Group by `customer_state` and `customer_city`
- Calculate mean, median, and percentiles (P25, P75, P90)
- Filter out cancelled/undelivered orders

**Expected Metrics:**
- Average delivery time (in days) by state
- Median delivery time (more robust to outliers)
- Standard deviation (delivery consistency)
- Percentage of orders delivered within 7, 14, 21, 30 days

**Visualizations:**
- Choropleth map of Brazil showing average delivery time by state
- Box plot comparing delivery time distributions across top 10 states
- Bar chart ranking states by average delivery time

---

#### 3. Delivery Delay Analysis
**Question:** Which regions/categories have the highest delivery delays relative to estimated delivery dates?

**Analysis Approach:**
- Calculate delay: `order_delivered_customer_date - order_estimated_delivery_date`
- Positive values = late delivery
- Join with products to analyze by category
- Join with customers to analyze by region

**Expected Metrics:**
- Average delay days by state/category
- Percentage of late deliveries by state/category
- Severity of delays (how many days late)
- On-time delivery rate (% delivered on or before estimated date)

**Visualizations:**
- Heatmap: states (rows) × categories (columns) showing average delay
- Pareto chart: top 10 state-category combinations by delay frequency
- Distribution plot: delay in days (negative = early, zero = on-time, positive = late)

---

#### 4. Review Scores vs Delivery Performance
**Question:** How do review scores vary by delivery delay (on-time vs late)?

**Analysis Approach:**
- Create delivery status categories: Early, On-Time, 1-3 days late, 4-7 days late, >7 days late
- Calculate average review score for each category
- Test statistical significance (t-test or ANOVA)
- Analyze review score distribution

**Expected Metrics:**
- Average review score by delivery status
- Percentage of 1-star, 5-star reviews by delivery status
- Correlation coefficient between delay days and review score
- Review comment sentiment by delivery status (if doing text analysis)

**Visualizations:**
- Grouped bar chart: average review score by delivery status
- Stacked bar chart: review score distribution (1-5 stars) for on-time vs late
- Scatter plot: delivery delay (x-axis) vs review score (y-axis) with trend line

---

#### 5. Category Risk Assessment
**Question:** Which product categories have the lowest average review scores and highest return-risk signals (e.g., low reviews + long delivery times)?

**Analysis Approach:**
- Calculate average review score by category
- Calculate average delivery time by category
- Create risk score: combine low satisfaction + long delivery
- Filter for categories with minimum sample size (e.g., >100 orders)

**Expected Metrics:**
- Average review score by category
- Average delivery time by category
- Risk score formula: `(5 - avg_review_score) × avg_delivery_days`
- Percentage of 1-2 star reviews by category

**Visualizations:**
- Scatter plot: avg review score (x) vs avg delivery time (y), sized by order volume
- Risk matrix: 2×2 grid (low/high satisfaction × fast/slow delivery)
- Table ranking categories by risk score with traffic light indicators

---

#### 6. Payment Method Analysis
**Question:** Which payment methods are most common and how do they relate to order value and order completion status?

**Analysis Approach:**
- Count orders by `payment_type`
- Calculate average payment value and total revenue by payment type
- Analyze payment installments pattern
- Cross-reference with `order_status` to identify completion rates

**Expected Metrics:**
- Order count and percentage by payment type
- Average order value by payment type
- Total revenue contribution by payment type
- Order completion rate by payment type
- Average installments by payment type

**Visualizations:**
- Pie chart: order distribution by payment method
- Bar chart: average order value by payment method
- Stacked bar: payment method usage over time
- Funnel chart: order status completion by payment method

---

### Additional Questions (Choose Minimum 4)

#### 7. Seller Delay Performance
**Question:** Which sellers contribute most to late deliveries (top 10 sellers by delay rate)?

**Analysis Approach:**
- Calculate late delivery rate per seller: `(late_orders / total_orders) × 100`
- Filter sellers with minimum order volume (e.g., >50 orders) to avoid statistical noise
- Rank by delay rate
- Calculate average delay days for each seller's late deliveries

**Expected Metrics:**
- Late delivery rate by seller (%)
- Average delay days by seller
- Total orders by seller
- Geographic location of worst-performing sellers

**Visualizations:**
- Horizontal bar chart: top 10 sellers by late delivery rate
- Scatter plot: total orders (x) vs late delivery rate (y)
- Tablee with seller details and performance metrics

---

#### 8. Seller Satisfaction Performance
**Question:** Which sellers contribute most to low review scores (top 10 sellers by avg review score and review volume)?

**Analysis Approach:**
- Join orders, order_items, and reviews to link sellers with review scores
- Calculate average review score per seller
- Filter for sellers with minimum review count (e.g., >30 reviews)
- Identify sellers with highest volume of 1-2 star reviews

**Expected Metrics:**
- Average review score by seller
- Number of reviews by seller
- Percentage of 1-2 star reviews by seller
- Seller location and performance correlation

**Visualizations:**
- Scatter plot: review count (x) vs avg review score (y)
- Bar chart: bottom 10 sellers by average review score
- Quadrant analysis: high volume/low satisfaction sellers

---

#### 9. Shipping Cost Analysis
**Question:** Do higher shipping costs correlate with faster delivery, slower delivery, or no pattern?

**Analysis Approach:**
- Calculate delivery time for each order
- Analyze `freight_value` distribution
- Create shipping cost bins (low, medium, high)
- Calculate correlation coefficient between freight value and delivery time
- Control for distance (use geolocation data)

**Expected Metrics:**
- Correlation coefficient (freight_value vs delivery_time)
- Average delivery time by freight value quartile
- Scatter plot slope and R² value
- Average freight value by delivery speed category

**Visualizations:**
- Scatter plot: freight value (x) vs delivery time (y) with trend line
- Box plot: delivery time distribution across freight value quartiles
- Heatmap: freight value bins × delivery time bins

**Statistical Tests:**
- Pearson correlation test
- Linear regression analysis
- Control for confounding variables (distance, product weight)

---

#### 10. Geographic Distribution Analysis
**Question:** What is the geographic distribution of orders (state/city heatmap), and where are the top revenue regions?

**Analysis Approach:**
- Aggregate orders and revenue by state and city
- Calculate revenue per capita (if population data available)
- Join with geolocation for mapping coordinates
- Identify geographic clusters of high-value customers

**Expected Metrics:**
- Order count by state/city
- Total revenue by state/city
- Average order value by state/city
- Market penetration (% of zip codes with orders)

**Visualizations:**
- Choropleth map: Brazil colored by order volume or revenue
- Bubble map: cities sized by revenue
- Bar chart: top 10 states and top 10 cities
- Treemap: hierarchical view (state → city → revenue)

---

#### 11. Category Revenue vs Satisfaction
**Question:** What are the top categories by revenue, and are they also top categories by satisfaction?

**Analysis Approach:**
- Calculate total revenue by category (sum of price + freight)
- Calculate average review score by category
- Create quadrant analysis: high/low revenue × high/low satisfaction
- Identify opportunity categories and problem categories

**Expected Metrics:**
- Revenue by category
- Order volume by category
- Average review score by category
- Revenue per order by category

**Visualizations:**
- Quadrant scatter plot: revenue (x) vs avg review score (y)
- Dual-axis chart: revenue bars with review score line overlay
- Table: side-by-side ranking of top categories by revenue and satisfaction

**Strategic Insights:**
- High revenue + high satisfaction = star products (maintain)
- High revenue + low satisfaction = fix urgently (revenue at risk)
- Low revenue + high satisfaction = growth opportunity
- Low revenue + low satisfaction = consider discontinuing

---

#### 12. Order Size Impact
**Question:** What is the relationship between number of items per order and delivery delay?

**Analysis Approach:**
- Count items per order from order_items table
- Calculate delivery delay for each order
- Analyze correlation and potential causal relationship
- Control for other factors (category, seller, region)

**Expected Metrics:**
- Average delivery time by items per order (1, 2, 3, 4, 5+ items)
- Percentage of late deliveries by order size
- Correlation coefficient
- Multi-item order frequency

**Visualizations:**
- Bar chart: average delivery time by number of items
- Box plot: delivery time distribution for different order sizes
- Stacked bar: on-time vs late by order size

---

#### 13. Seasonality Analysis
**Question:** What is the seasonality of purchases (peak months) and does delivery performance degrade during peaks?

**Analysis Approach:**
- Extract month, quarter, and day-of-week from purchase timestamp
- Identify peak shopping periods (Black Friday, Christmas, etc.)
- Compare delivery metrics during peak vs non-peak periods
- Analyze capacity constraints

**Expected Metrics:**
- Order volume by month
- Average delivery time by month
- On-time delivery rate by month
- Peak period identification (top 5 days/weeks)

**Visualizations:**
- Line chart: monthly order volume with delivery performance overlay
- Heatmap: day of week × month showing order patterns
- Before/after comparison: peak period delivery metrics

**Additional Analysis:**
- Early/on-time/late distribution over time
- Trend analysis: has delivery performance improved or degraded?
- Capacity planning insights

---

## 🧹 Data Quality Issues & Resolution Strategies

### 1. Missing Values

#### Orders Table
**Issue:** 
- `order_approved_at`: 160 missing (0.2%)
- `order_delivered_carrier_date`: 1,783 missing (1.8%)
- `order_delivered_customer_date`: 2,965 missing (3.0%)

**Investigation Needed:**
- Cross-reference with `order_status`
- Check if missing deliveries correlate with cancelled/pending orders
- Verify if missing approved_at indicates payment issues

**Resolution Strategy:**
```python
# Strategy 1: Analyze missing pattern
missing_delivery = orders[orders['order_delivered_customer_date'].isnull()]
print(missing_delivery['order_status'].value_counts())

# Strategy 2: Exclude or flag for analysis
# For delivery time analysis: exclude records with missing delivery dates
# For order count analysis: include all orders
# For completion rate analysis: missing = not delivered

# Strategy 3: Document assumptions
# Assumption: Missing delivery date means order not completed
# Create is_delivered flag for clarity
orders['is_delivered'] = orders['order_delivered_customer_date'].notnull()
```

---

#### Products Table
**Issue:**
- `product_category_name`: 610 missing (1.9%)
- `product_name_lenght`: 610 missing (1.9%)
- `product_description_lenght`: 610 missing (1.9%)
- `product_photos_qty`: 610 missing (1.9%)
- Dimension columns (`weight_g`, `length_cm`, etc.): 2 missing (0.0%)

**Pattern Recognition:**
- Same 610 products missing all metadata → likely incomplete product listings
- 1.9% of products, but what % of revenue?

**Resolution Strategy:**
```python
# Strategy 1: Impact assessment
products_missing_category = products[products['product_category_name'].isnull()]
# Join with order_items to calculate revenue impact
revenue_impact = order_items.merge(products_missing_category, on='product_id')
# If <1% of revenue, can exclude or create "Unknown" category

# Strategy 2: Create "Uncategorized" category
products['product_category_name'].fillna('uncategorized', inplace=True)

# Strategy 3: For dimension attributes, consider:
# - Median imputation by category
# - Exclude from weight/size analysis
# - Flag for manual review

# Strategy 4: Check if English translation exists
missing_translation = set(products['product_category_name']) - set(category_translation['product_category_name'])
```

---

#### Reviews Table
**Issue:**
- `review_comment_title`: 87,656 missing (88.3%)
- `review_comment_message`: 58,247 missing (58.7%)

**Context:**
- This is expected user behavior (optional comment fields)
- Not missing data, but intentionally left blank
- Focus on `review_score` which has no missing values

**Resolution Strategy:**
```python
# No imputation needed - missing comments are valid
# For text analysis:
# - Analyze only reviews with comments
# - Calculate % with comments as engagement metric
# - Compare avg score for reviews with/without comments

engagement_rate = (reviews['review_comment_message'].notnull().sum() / len(reviews)) * 100
# Hypothesis: Users who leave comments may have stronger opinions (very happy or very unhappy)
```

---

### 2. Duplicate Records

#### Geolocation Table
**Issue:** 261,831 duplicate rows (26.2% of 1,000,163 records)

**Investigation:**
```python
# Check what defines a duplicate
geolocation.duplicated(subset=['geolocation_zip_code_prefix']).sum()
# vs
geolocation.duplicated().sum()

# Hypothesis: Same zip code has multiple lat/lng coordinates
# This could represent:
# 1. Geographic area of the zip code (multiple points)
# 2. Data quality issues (incorrect geocoding)
# 3. Multiple addresses within same zip
```

**Resolution Strategy:**
```python
# Strategy 1: Keep all records (represents geographic spread)
# Use case: Calculate average distance between customer and seller

# Strategy 2: Aggregate to centroid per zip code
geo_centroids = geolocation.groupby('geolocation_zip_code_prefix').agg({
    'geolocation_lat': 'mean',
    'geolocation_lng': 'mean',
    'geolocation_city': 'first',  # or most frequent
    'geolocation_state': 'first'
}).reset_index()

# Strategy 3: Keep unique zip-city-state combinations
geo_unique = geolocation.drop_duplicates(
    subset=['geolocation_zip_code_prefix', 'geolocation_city', 'geolocation_state']
)

# Recommendation: Use Strategy 2 (centroid) for distance calculations
```

---

### 3. Data Type Conversions

**Issue:** All timestamp columns stored as `object` instead of `datetime64`

**Columns to Convert:**
- `order_purchase_timestamp`
- `order_approved_at`
- `order_delivered_carrier_date`
- `order_delivered_customer_date`
- `order_estimated_delivery_date`
- `shipping_limit_date`
- `review_creation_date`
- `review_answer_timestamp`

**Resolution:**
```python
# Conversion function
def convert_timestamps(df, columns):
    for col in columns:
        if col in df.columns:
            df[col] = pd.to_datetime(df[col], errors='coerce')
            # errors='coerce' converts invalid parsing to NaT
    return df

# Apply to each dataframe
timestamp_cols_orders = [
    'order_purchase_timestamp',
    'order_approved_at',
    'order_delivered_carrier_date',
    'order_delivered_customer_date',
    'order_estimated_delivery_date'
]
orders = convert_timestamps(orders, timestamp_cols_orders)

# Verify conversion
orders.dtypes
orders['order_purchase_timestamp'].describe()  # Should show datetime statistics
```

---

### 4. Outlier Detection & Handling

#### Price and Freight Analysis
```python
# Identify outliers using IQR method
def detect_outliers_iqr(df, column):
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    
    outliers = df[(df[column] < lower_bound) | (df[column] > upper_bound)]
    return outliers, lower_bound, upper_bound

# Check price outliers
price_outliers, price_lower, price_upper = detect_outliers_iqr(order_items, 'price')
print(f"Price outliers: {len(price_outliers)} ({len(price_outliers)/len(order_items)*100:.2f}%)")
print(f"Price range: {price_lower:.2f} to {price_upper:.2f}")

# Check freight outliers
freight_outliers, freight_lower, freight_upper = detect_outliers_iqr(order_items, 'freight_value')

# Strategy: Don't remove outliers, but flag them for investigation
# Luxury items may legitimately cost 10x median price
```

#### Delivery Time Analysis
```python
# Calculate delivery time
orders['delivery_time_days'] = (
    orders['order_delivered_customer_date'] - 
    orders['order_purchase_timestamp']
).dt.total_seconds() / (24 * 3600)

# Check for impossible values
negative_delivery = orders[orders['delivery_time_days'] < 0]
extreme_delivery = orders[orders['delivery_time_days'] > 200]  # >6 months

# Investigation needed for:
# - Negative delivery times (data entry errors)
# - Extremely long delivery (>60 days)

# Resolution: Cap or exclude from specific analyses
orders_clean = orders[
    (orders['delivery_time_days'] >= 0) & 
    (orders['delivery_time_days'] <= 200)
]
```

---

### 5. Join Strategy & Data Integration

#### Primary Master Table Creation

**Objective:** Create analysis-ready master table with one row per order

```python
# Step 1: Start with orders as base
master = orders.copy()

# Step 2: Add customer information (1:1 join)
master = master.merge(
    customers,
    on='customer_id',
    how='left',
    validate='1:1'
)

# Step 3: Aggregate order items (Many:1 join)
order_summary = order_items.groupby('order_id').agg({
    'order_item_id': 'count',  # number of items
    'price': 'sum',  # total product value
    'freight_value': 'sum',  # total shipping cost
    'product_id': lambda x: list(x),  # list of products (optional)
    'seller_id': lambda x: list(x)  # list of sellers (optional)
}).reset_index()

order_summary.columns = [
    'order_id', 'total_items', 'total_price', 
    'total_freight', 'product_ids', 'seller_ids'
]

master = master.merge(
    order_summary,
    on='order_id',
    how='left',
    validate='1:1'
)

# Step 4: Aggregate payments (Many:1 join)
payment_summary = payments.groupby('order_id').agg({
    'payment_value': 'sum',
    'payment_type': lambda x: ', '.join(x.unique()),
    'payment_installments': 'max'
}).reset_index()

payment_summary.columns = [
    'order_id', 'total_payment_value', 
    'payment_methods', 'max_installments'
]

master = master.merge(
    payment_summary,
    on='order_id',
    how='left',
    validate='1:1'
)

# Step 5: Add reviews (1:0..1 join)
reviews_subset = order_reviews[[
    'order_id', 'review_score', 
    'review_comment_title', 'review_comment_message'
]]

master = master.merge(
    reviews_subset,
    on='order_id',
    how='left',
    validate='1:1'
)

# Step 6: Add customer geolocation (for distance calculations)
customer_geo = geolocation.groupby('geolocation_zip_code_prefix').agg({
    'geolocation_lat': 'mean',
    'geolocation_lng': 'mean'
}).reset_index()

customer_geo.columns = [
    'customer_zip_code_prefix', 'customer_lat', 'customer_lng'
]

master = master.merge(
    customer_geo,
    on='customer_zip_code_prefix',
    how='left',
    validate='m:1'
)

# Step 7: Validate master table
print(f"Master table shape: {master.shape}")
print(f"Expected rows: {len(orders)}")
print(f"Actual rows: {len(master)}")
assert len(master) == len(orders), "Row count mismatch in master table!"
```

#### Secondary Analysis Tables

**Product-Level Analysis:**
```python
# For category-level analysis
product_orders = order_items.merge(
    products,
    on='product_id',
    how='left'
).merge(
    category_translation,
    on='product_category_name',
    how='left'
).merge(
    orders[['order_id', 'order_delivered_customer_date', 
            'order_estimated_delivery_date', 'customer_state']],
    on='order_id',
    how='left'
).merge(
    order_reviews[['order_id', 'review_score']],
    on='order_id',
    how='left'
)
```

**Seller-Level Analysis:**
```python
# For seller performance analysis
seller_orders = order_items.merge(
    sellers,
    on='seller_id',
    how='left'
).merge(
    orders[['order_id', 'order_delivered_customer_date', 
            'order_estimated_delivery_date', 'customer_state']],
    on='order_id',
    how='left'
).merge(
    order_reviews[['order_id', 'review_score']],
    on='order_id',
    how='left'
)
```

---

### 6. Feature Engineering

#### Calculated Metrics

```python
# Delivery performance metrics
master['delivery_time_days'] = (
    master['order_delivered_customer_date'] - 
    master['order_purchase_timestamp']
).dt.total_seconds() / (24 * 3600)

master['estimated_delivery_time_days'] = (
    master['order_estimated_delivery_date'] - 
    master['order_purchase_timestamp']
).dt.total_seconds() / (24 * 3600)

master['delivery_delay_days'] = (
    master['order_delivered_customer_date'] - 
    master['order_estimated_delivery_date']
).dt.total_seconds() / (24 * 3600)

# Delivery status categories
def categorize_delivery(delay_days):
    if pd.isna(delay_days):
        return 'Not Delivered'
    elif delay_days < -2:
        return 'Very Early (>2 days)'
    elif delay_days < 0:
        return 'Early'
    elif delay_days == 0:
        return 'On Time'
    elif delay_days <= 3:
        return 'Slightly Late (1-3 days)'
    elif delay_days <= 7:
        return 'Late (4-7 days)'
    else:
        return 'Very Late (>7 days)'

master['delivery_status'] = master['delivery_delay_days'].apply(categorize_delivery)

# On-time delivery flag
master['is_on_time'] = master['delivery_delay_days'] <= 0
master['is_late'] = master['delivery_delay_days'] > 0

# Order value metrics
master['order_total'] = master['total_price'] + master['total_freight']
master['avg_item_price'] = master['total_price'] / master['total_items']
master['freight_percentage'] = (master['total_freight'] / master['order_total']) * 100

# Temporal features
master['purchase_year'] = master['order_purchase_timestamp'].dt.year
master['purchase_month'] = master['order_purchase_timestamp'].dt.month
master['purchase_quarter'] = master['order_purchase_timestamp'].dt.quarter
master['purchase_day_of_week'] = master['order_purchase_timestamp'].dt.dayofweek
master['purchase_hour'] = master['order_purchase_timestamp'].dt.hour
master['purchase_month_name'] = master['order_purchase_timestamp'].dt.month_name()

# Review metrics
master['has_review'] = master['review_score'].notnull()
master['has_comment'] = master['review_comment_message'].notnull()
master['is_positive_review'] = master['review_score'] >= 4
master['is_negative_review'] = master['review_score'] <= 2

# Payment metrics
master['uses_installments'] = master['max_installments'] > 1
master['installment_value'] = master['total_payment_value'] / master['max_installments']

# Order complexity
master['is_multi_item'] = master['total_items'] > 1
master['order_size_category'] = pd.cut(
    master['total_items'],
    bins=[0, 1, 2, 5, float('inf')],
    labels=['Single Item', '2 Items', '3-5 Items', '6+ Items']
)
```

#### Distance Calculation (Advanced)

```python
from math import radians, cos, sin, asin, sqrt

def haversine(lon1, lat1, lon2, lat2):
    """
    Calculate distance between two points on earth in kilometers
    """
    lon1, lat1, lon2, lat2 = map(radians, [lon1, lat1, lon2, lat2])
    dlon = lon2 - lon1 
    dlat = lat2 - lat1 
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    c = 2 * asin(sqrt(a)) 
    r = 6371  # Earth radius in kilometers
    return c * r

# Apply to calculate customer-seller distance
# (Requires joining seller geolocation data)
```

---

## 📈 Power BI Dashboard Design

### Data Modeling in Power BI

#### Relationships
```
customers (1) ←→ (*) orders
orders (1) ←→ (*) order_items
order_items (*) ←→ (1) products
order_items (*) ←→ (1) sellers
orders (1) ←→ (*) payments
orders (1) ←→ (1) reviews
products (*) ←→ (1) category_translation
```

#### DAX Measures - Core Calculations

```dax
// Revenue Metrics
Total Revenue = SUM(payments[payment_value])

Total Orders = DISTINCTCOUNT(orders[order_id])

Average Order Value = DIVIDE([Total Revenue], [Total Orders])

Revenue Growth MoM = 
VAR CurrentRevenue = [Total Revenue]
VAR PreviousRevenue = CALCULATE([Total Revenue], DATEADD('Date'[Date], -1, MONTH))
RETURN DIVIDE(CurrentRevenue - PreviousRevenue, PreviousRevenue)

// Delivery Metrics
Average Delivery Time = 
AVERAGE(
    DATEDIFF(
        orders[order_purchase_timestamp],
        orders[order_delivered_customer_date],
        DAY
    )
)

On-Time Delivery Rate = 
VAR OnTimeOrders = 
    CALCULATE(
        COUNTROWS(orders),
        orders[order_delivered_customer_date] <= orders[order_estimated_delivery_date]
    )
VAR TotalDelivered = 
    CALCULATE(
        COUNTROWS(orders),
        NOT(ISBLANK(orders[order_delivered_customer_date]))
    )
RETURN DIVIDE(OnTimeOrders, TotalDelivered)

Late Delivery Rate = 1 - [On-Time Delivery Rate]

Average Delay Days = 
AVERAGE(
    DATEDIFF(
        orders[order_estimated_delivery_date],
        orders[order_delivered_customer_date],
        DAY
    )
)

// Review Metrics
Average Review Score = AVERAGE(reviews[review_score])

Positive Review Rate = 
DIVIDE(
    CALCULATE(COUNTROWS(reviews), reviews[review_score] >= 4),
    COUNTROWS(reviews)
)

Review Response Rate = 
DIVIDE(
    COUNTROWS(reviews),
    DISTINCTCOUNT(orders[order_id])
)

// Customer Metrics
Total Customers = DISTINCTCOUNT(customers[customer_unique_id])

Repeat Customer Rate = 
VAR RepeatCustomers = 
    CALCULATE(
        DISTINCTCOUNT(customers[customer_unique_id]),
        FILTER(
            VALUES(customers[customer_unique_id]),
            CALCULATE(DISTINCTCOUNT(orders[order_id])) > 1
        )
    )
RETURN DIVIDE(RepeatCustomers, [Total Customers])

// Seller Metrics
Total Sellers = DISTINCTCOUNT(sellers[seller_id])

Seller Efficiency Score = 
[On-Time Delivery Rate] * 0.5 + 
([Average Review Score] / 5) * 0.5
```

---

### Page 1: Executive Overview

#### Layout Structure & Components

**KPI Cards (Top Row):**
1. **Total Orders**
   - Value: 99,441
   - Trend indicator: MoM growth %
   - Conditional formatting: Green if growth > 0

2. **Total Revenue** 
   - Value: Sum of payment_value
   - Format: Currency (R$)
   - Comparison to previous period

3. **Average Delivery Time**
   - Value: Average days from purchase to delivery
   - Benchmark: Industry standard or target
   - Color: Red if > 15 days, Yellow if 10-15, Green if < 10

4. **On-Time Delivery Rate**
   - Value: % of orders delivered by estimated date
   - Target line at 95%
   - Historical trend sparkline

5. **Average Review Score**
   - Value: 1-5 star rating
   - Visual: Star icons
   - Comparison to target (4.0+)

**Monthly Trend Chart:**
- **Type:** Dual-axis line chart
- **Primary Axis:** Order count (bars)
- **Secondary Axis:** Revenue (line)
- **X-Axis:** Month-Year
- **Features:**
  - Tooltips showing exact values
  - Trend lines for both metrics
  - Highlight peak months
  - Date slicer for filtering

**Key Insights Box:**
- **Type:** Text box / Card visual
- **Content Format:**
  ```
  📊 KEY INSIGHTS (Updated: [Date])
  
  ✓ 92.3% on-time delivery rate (↑2.1% vs Q3)
  ⚠ SP state shows 15% higher delays than average
  ✓ Credit card orders have 8% higher completion rate
  ⚠ Electronics category has lowest review scores (3.6)
  ✓ Revenue grew 12% MoM, driven by increased AOV
  ```

**Filters/Slicers:**
- Date range (month/quarter/year)
- State dropdown
- Order status multi-select
- Product category

**Interactivity:**
- Click on month → filters all other pages
- Click on KPI → drills to relevant detail page
- Hover tooltips showing detailed metrics
---

### Page 2: Delivery & Logistics Performance

#### Primary Visuals

**1. Delivery Time Distribution**
- **Type:** Histogram with normal curve overlay
- **X-Axis:** Delivery time in days (bins: 0-5, 6-10, 11-15, 16-20, 21-30, 30+)
- **Y-Axis:** Number of orders
- **Statistical Overlays:**
  - Mean line (vertical red line)
  - Median line (vertical blue line)
  - Standard deviation bands
- **Insights Box:** Shows % of orders in each bin
- **Conditional Formatting:** Color bins based on acceptable/unacceptable delivery times

**2. Geographic Delivery Performance Map**
- **Type:** Filled (Choropleth) Map of Brazil
- **Geography Level:** State
- **Color Scale:** Average delivery delay (days)
  - Green: Early/on-time (< 0 days)
  - Yellow: Slightly late (0-3 days)
  - Orange: Late (3-7 days)
  - Red: Very late (> 7 days)
- **Tooltips:** State name, avg delivery time, on-time rate, total orders
- **Drill-through:** Click state → see city-level breakdown

**3. Late Delivery Rate Ranking**
- **Type:** Horizontal bar chart
- **Data:** Top 10 states by late delivery percentage
- **Bars:** Sorted descending
- **Color:** Gradient from red (worst) to orange (better)
- **Data Labels:** Percentage + order count
- **Benchmark Line:** National average late delivery rate

**4. Delivery Performance Trend Over Time**
- **Type:** Area chart or line chart with shaded regions
- **X-Axis:** Month
- **Y-Axis:** Percentage (0-100%)
- **Series:**
  - % Early deliveries (bottom, green)
  - % On-time deliveries (middle, blue)
  - % Late deliveries (top, red)
- **Stacked:** Shows composition adds to 100%
- **Annotations:** Mark significant events (holidays, peak seasons)

**5. Delivery Time by Category**
- **Type:** Box plot or violin plot
- **Categories:** Top 10 product categories by order volume
- **Shows:** Min, Q1, Median, Q3, Max delivery times
- **Outliers:** Displayed as individual points
- **Allows:** Quick comparison of delivery time variability across categories

**Filters/Slicers:**
- Date range
- State (multi-select)
- Product category
- Delivery status (early/on-time/late)
- Order value range

**Calculated Metrics (Displayed as Cards):**
- Average delay: "X.X days"
- Worst performing region: "State: XX (YY% late)"
- Best performing region: "State: ZZ (AA% on-time)"
- Trend indicator: "↑ Delays increasing 2.3% MoM"

---

### Page 3: Customer Experience & Satisfaction

#### Primary Visuals

**1. Review Score Distribution**
- **Type:** Stacked bar chart or donut chart
- **Categories:** 1-star to 5-star
- **Values:** Count and percentage of each rating
- **Visual Design:**
  - 5-star: Dark green
  - 4-star: Light green
  - 3-star: Yellow
  - 2-star: Orange
  - 1-star: Red
- **Summary Metrics:**
  - Net Promoter Score equivalent (% 4-5 stars - % 1-2 stars)
  - Customer Satisfaction Score (CSAT)

**2. Review Score vs Delivery Performance**
- **Type:** Grouped column chart
- **X-Axis:** Delivery status categories
  - Very Early (>2 days early)
  - Early (1-2 days early)
  - On-Time
  - Slightly Late (1-3 days)
  - Late (4-7 days)
  - Very Late (>7 days)
- **Y-Axis:** Average review score
- **Bars:** Color-coded by delivery status
- **Error Bars:** Show confidence intervals
- **Trend Line:** Shows correlation between delay and satisfaction
- **Statistical Note:** Include correlation coefficient (r value)

**3. Category Performance Matrix**
- **Type:** Scatter plot (bubble chart)
- **X-Axis:** Average review score
- **Y-Axis:** Order volume
- **Bubble Size:** Total revenue
- **Color:** Delivery performance (green = on-time, red = delayed)
- **Quadrants:**
  - Top-right: High volume + High satisfaction (Stars)
  - Top-left: High volume + Low satisfaction (Fix Urgently)
  - Bottom-right: Low volume + High satisfaction (Opportunity)
  - Bottom-left: Low volume + Low satisfaction (Discontinue)
- **Labels:** Product category names on bubbles
- **Minimum Filter:** Show only categories with >100 reviews

**4. Worst Performing Categories**
- **Type:** Table with conditional formatting
- **Columns:**
  - Category Name
  - Avg Review Score (color scale: red to green)
  - Total Reviews
  - % Negative Reviews (1-2 stars)
  - Avg Delivery Time
  - On-Time Rate
- **Sort:** By average review score (ascending)
- **Filter:** Minimum 50 reviews to avoid statistical noise
- **Top N:** Show bottom 10 categories

**5. Seller Performance - Problem Sellers**
- **Type:** Matrix or table
- **Columns:**
  - Seller ID
  - Total Orders
  - Avg Review Score
  - Late Delivery %
  - Total Revenue
  - Risk Score (composite metric)
- **Conditional Formatting:**
  - Red: Avg review < 3.0 OR late delivery > 20%
  - Yellow: Avg review 3.0-3.5 OR late delivery 10-20%
  - Green: Avg review > 4.0 AND late delivery < 10%
- **Filter:** Sellers with minimum 30 orders
- **Sort:** By risk score desc# Olist E-Commerce Data Analysis Project

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

### Dataset Structure & Relationships

```
customers
├── customer_id (PK)
├── customer_unique_id
├── customer_zip_code_prefix
├── customer_city
└── customer_state

orders
├── order_id (PK)
├── customer_id (FK → customers.customer_id)
├── order_status
├── order_purchase_timestamp
├── order_approved_at
├── order_delivered_carrier_date
├── order_delivered_customer_date
└── order_estimated_delivery_date

order_items
├── order_id (FK → orders.order_id)
├── order_item_id
├── product_id (FK → products.product_id)
├── seller_id (FK → sellers.seller_id)
├── shipping_limit_date
├── price
└── freight_value

products
├── product_id (PK)
├── product_category_name (FK → category_translation)
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
├── order_id (FK → orders.order_id)
├── payment_sequential
├── payment_type
├── payment_installments
└── payment_value

order_reviews
├── review_id (PK)
├── order_id (FK → orders.order_id)
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
├── product_category_name (PK)
└── product_category_name_english
```

### Key Relationships Explained

**1. Customer to Order (1:1)**
- Each order belongs to exactly one customer
- 99,441 customers mapped to 99,441 orders
- However, 96,096 unique `customer_unique_id` suggests ~3,345 repeat customers
- **Analysis implication:** Can identify repeat purchase behavior and customer lifetime value

**2. Order to Order Items (1:Many)**
- Each order can have multiple line items
- 112,650 order items across 98,666 unique orders
- Average: 1.13 items per order
- **Analysis implication:** Most orders are single-item; multi-item orders may have different delivery/satisfaction patterns

**3. Order to Payment (1:Many)**
- Some orders have multiple payment transactions
- 103,886 payment records for 99,440 unique orders
- Average: 1.04 payments per order
- **Analysis implication:** Split payments exist; need aggregation for total order value

**4. Order to Review (1:0..1)**
- Not all orders have reviews
- 99,224 reviews for 99,441 orders (217 orders without reviews)
- 99.8% review rate
- **Analysis implication:** High engagement but missing reviews may correlate with order status

**5. Product to Category (Many:1)**
- 32,951 products across 73 categories
- 610 products (1.9%) have no category assigned
- **Analysis implication:** Uncategorized products need handling strategy

**6. Geographic Relationships**
- Customers, sellers, and geolocation linked via zip code prefix
- 19,015 unique zip codes in geolocation vs 14,994 customer zips
- **Analysis implication:** Can calculate distance between customer and seller

---

## 🎯 Analytical Questions

### Core Questions (Mandatory - All 6)

#### 1. Temporal Revenue Analysis
**Question:** How do order counts and revenue change over time (monthly/quarterly)?

**Analysis Approach:**
- Extract month/quarter from `order_purchase_timestamp`
- Aggregate order counts and sum of payment values
- Calculate growth rates (MoM, QoQ)
- Identify seasonal patterns and trends

**Expected Metrics:**
- Monthly order volume
- Monthly revenue (sum of payment_value)
- Average order value (AOV) by month
- Revenue growth rate
- Order count growth rate

**Visualizations:**
- Dual-axis line chart (orders vs revenue over time)
- Bar chart for quarterly comparisons
- Heatmap showing day-of-week and hour-of-day patterns

---

#### 2. Delivery Performance by Region
**Question:** What is the average delivery time and how does it vary by region/state/city?

**Analysis Approach:**
- Calculate actual delivery time: `order_delivered_customer_date - order_purchase_timestamp`
- Group by `customer_state` and `customer_city`
- Calculate mean, median, and percentiles (P25, P75, P90)
- Filter out cancelled/undelivered orders

**Expected Metrics:**
- Average delivery time (in days) by state
- Median delivery time (more robust to outliers)
- Standard deviation (delivery consistency)
- Percentage of orders delivered within 7, 14, 21, 30 days

**Visualizations:**
- Choropleth map of Brazil showing average delivery time by state
- Box plot comparing delivery time distributions across top 10 states
- Bar chart ranking states by average delivery time

---

#### 3. Delivery Delay Analysis
**Question:** Which regions/categories have the highest delivery delays relative to estimated delivery dates?

**Analysis Approach:**
- Calculate delay: `order_delivered_customer_date - order_estimated_delivery_date`
- Positive values = late delivery
- Join with products to analyze by category
- Join with customers to analyze by region

**Expected Metrics:**
- Average delay days by state/category
- Percentage of late deliveries by state/category
- Severity of delays (how many days late)
- On-time delivery rate (% delivered on or before estimated date)

**Visualizations:**
- Heatmap: states (rows) × categories (columns) showing average delay
- Pareto chart: top 10 state-category combinations by delay frequency
- Distribution plot: delay in days (negative = early, zero = on-time, positive = late)

---

#### 4. Review Scores vs Delivery Performance
**Question:** How do review scores vary by delivery delay (on-time vs late)?

**Analysis Approach:**
- Create delivery status categories: Early, On-Time, 1-3 days late, 4-7 days late, >7 days late
- Calculate average review score for each category
- Test statistical significance (t-test or ANOVA)
- Analyze review score distribution

**Expected Metrics:**
- Average review score by delivery status
- Percentage of 1-star, 5-star reviews by delivery status
- Correlation coefficient between delay days and review score
- Review comment sentiment by delivery status (if doing text analysis)

**Visualizations:**
- Grouped bar chart: average review score by delivery status
- Stacked bar chart: review score distribution (1-5 stars) for on-time vs late
- Scatter plot: delivery delay (x-axis) vs review score (y-axis) with trend line

---

#### 5. Category Risk Assessment
**Question:** Which product categories have the lowest average review scores and highest return-risk signals (e.g., low reviews + long delivery times)?

**Analysis Approach:**
- Calculate average review score by category
- Calculate average delivery time by category
- Create risk score: combine low satisfaction + long delivery
- Filter for categories with minimum sample size (e.g., >100 orders)

**Expected Metrics:**
- Average review score by category
- Average delivery time by category
- Risk score formula: `(5 - avg_review_score) × avg_delivery_days`
- Percentage of 1-2 star reviews by category

**Visualizations:**
- Scatter plot: avg review score (x) vs avg delivery time (y), sized by order volume
- Risk matrix: 2×2 grid (low/high satisfaction × fast/slow delivery)
- Table ranking categories by risk score with traffic light indicators

---

#### 6. Payment Method Analysis
**Question:** Which payment methods are most common and how do they relate to order value and order completion status?

**Analysis Approach:**
- Count orders by `payment_type`
- Calculate average payment value and total revenue by payment type
- Analyze payment installments pattern
- Cross-reference with `order_status` to identify completion rates

**Expected Metrics:**
- Order count and percentage by payment type
- Average order value by payment type
- Total revenue contribution by payment type
- Order completion rate by payment type
- Average installments by payment type

**Visualizations:**
- Pie chart: order distribution by payment method
- Bar chart: average order value by payment method
- Stacked bar: payment method usage over time
- Funnel chart: order status completion by payment method

---

### Additional Questions (Choose Minimum 4)

#### 7. Seller Delay Performance
**Question:** Which sellers contribute most to late deliveries (top 10 sellers by delay rate)?

**Analysis Approach:**
- Calculate late delivery rate per seller: `(late_orders / total_orders) × 100`
- Filter sellers with minimum order volume (e.g., >50 orders) to avoid statistical noise
- Rank by delay rate
- Calculate average delay days for each seller's late deliveries

**Expected Metrics:**
- Late delivery rate by seller (%)
- Average delay days by seller
- Total orders by seller
- Geographic location of worst-performing sellers

**Visualizations:**
- Horizontal bar chart: top 10 sellers by late delivery rate
- Scatter plot: total orders (x) vs late delivery rate (y)
- Table with seller details and performance metrics

---

#### 8. Seller Satisfaction Performance
**Question:** Which sellers contribute most to low review scores (top 10 sellers by avg review score and review volume)?

**Analysis Approach:**
- Join orders, order_items, and reviews to link sellers with review scores
- Calculate average review score per seller
- Filter for sellers with minimum review count (e.g., >30 reviews)
- Identify sellers with highest volume of 1-2 star reviews

**Expected Metrics:**
- Average review score by seller
- Number of reviews by seller
- Percentage of 1-2 star reviews by seller
- Seller location and performance correlation

**Visualizations:**
- Scatter plot: review count (x) vs avg review score (y)
- Bar chart: bottom 10 sellers by average review score
- Quadrant analysis: high volume/low satisfaction sellers

---

#### 9. Shipping Cost Analysis
**Question:** Do higher shipping costs correlate with faster delivery, slower delivery, or no pattern?

**Analysis Approach:**
- Calculate delivery time for each order
- Analyze `freight_value` distribution
- Create shipping cost bins (low, medium, high)
- Calculate correlation coefficient between freight value and delivery time
- Control for distance (use geolocation data)

**Expected Metrics:**
- Correlation coefficient (freight_value vs delivery_time)
- Average delivery time by freight value quartile
- Scatter plot slope and R² value
- Average freight value by delivery speed category

**Visualizations:**
- Scatter plot: freight value (x) vs delivery time (y) with trend line
- Box plot: delivery time distribution across freight value quartiles
- Heatmap: freight value bins × delivery time bins

**Statistical Tests:**
- Pearson correlation test
- Linear regression analysis
- Control for confounding variables (distance, product weight)

---

#### 10. Geographic Distribution Analysis
**Question:** What is the geographic distribution of orders (state/city heatmap), and where are the top revenue regions?

**Analysis Approach:**
- Aggregate orders and revenue by state and city
- Calculate revenue per capita (if population data available)
- Join with geolocation for mapping coordinates
- Identify geographic clusters of high-value customers

**Expected Metrics:**
- Order count by state/city
- Total revenue by state/city
- Average order value by state/city
- Market penetration (% of zip codes with orders)

**Visualizations:**
- Choropleth map: Brazil colored by order volume or revenue
- Bubble map: cities sized by revenue
- Bar chart: top 10 states and top 10 cities
- Treemap: hierarchical view (state → city → revenue)

---

#### 11. Category Revenue vs Satisfaction
**Question:** What are the top categories by revenue, and are they also top categories by satisfaction?

**Analysis Approach:**
- Calculate total revenue by category (sum of price + freight)
- Calculate average review score by category
- Create quadrant analysis: high/low revenue × high/low satisfaction
- Identify opportunity categories and problem categories

**Expected Metrics:**
- Revenue by category
- Order volume by category
- Average review score by category
- Revenue per order by category

**Visualizations:**
- Quadrant scatter plot: revenue (x) vs avg review score (y)
- Dual-axis chart: revenue bars with review score line overlay
- Table: side-by-side ranking of top categories by revenue and satisfaction

**Strategic Insights:**
- High revenue + high satisfaction = star products (maintain)
- High revenue + low satisfaction = fix urgently (revenue at risk)
- Low revenue + high satisfaction = growth opportunity
- Low revenue + low satisfaction = consider discontinuing

---

#### 12. Order Size Impact
**Question:** What is the relationship between number of items per order and delivery delay?

**Analysis Approach:**
- Count items per order from order_items table
- Calculate delivery delay for each order
- Analyze correlation and potential causal relationship
- Control for other factors (category, seller, region)

**Expected Metrics:**
- Average delivery time by items per order (1, 2, 3, 4, 5+ items)
- Percentage of late deliveries by order size
- Correlation coefficient
- Multi-item order frequency

**Visualizations:**
- Bar chart: average delivery time by number of items
- Box plot: delivery time distribution for different order sizes
- Stacked bar: on-time vs late by order size

---

#### 13. Seasonality Analysis
**Question:** What is the seasonality of purchases (peak months) and does delivery performance degrade during peaks?

**Analysis Approach:**
- Extract month, quarter, and day-of-week from purchase timestamp
- Identify peak shopping periods (Black Friday, Christmas, etc.)
- Compare delivery metrics during peak vs non-peak periods
- Analyze capacity constraints

**Expected Metrics:**
- Order volume by month
- Average delivery time by month
- On-time delivery rate by month
- Peak period identification (top 5 days/weeks)

**Visualizations:**
- Line chart: monthly order volume with delivery performance overlay
- Heatmap: day of week × month showing order patterns
- Before/after comparison: peak period delivery metrics

**Additional Analysis:**
- Early/on-time/late distribution over time
- Trend analysis: has delivery performance improved or degraded?
- Capacity planning insights

---

## 🧹 Data Quality Issues & Resolution Strategies

### 1. Missing Values

#### Orders Table
**Issue:** 
- `order_approved_at`: 160 missing (0.2%)
- `order_delivered_carrier_date`: 1,783 missing (1.8%)
- `order_delivered_customer_date`: 2,965 missing (3.0%)

**Investigation Needed:**
- Cross-reference with `order_status`
- Check if missing deliveries correlate with cancelled/pending orders
- Verify if missing approved_at indicates payment issues

**Resolution Strategy:**
```python
# Strategy 1: Analyze missing pattern
missing_delivery = orders[orders['order_delivered_customer_date'].isnull()]
print(missing_delivery['order_status'].value_counts())

# Strategy 2: Exclude or flag for analysis
# For delivery time analysis: exclude records with missing delivery dates
# For order count analysis: include all orders
# For completion rate analysis: missing = not delivered

# Strategy 3: Document assumptions
# Assumption: Missing delivery date means order not completed
# Create is_delivered flag for clarity
orders['is_delivered'] = orders['order_delivered_customer_date'].notnull()
```

---

#### Products Table
**Issue:**
- `product_category_name`: 610 missing (1.9%)
- `product_name_lenght`: 610 missing (1.9%)
- `product_description_lenght`: 610 missing (1.9%)
- `product_photos_qty`: 610 missing (1.9%)
- Dimension columns (`weight_g`, `length_cm`, etc.): 2 missing (0.0%)

**Pattern Recognition:**
- Same 610 products missing all metadata → likely incomplete product listings
- 1.9% of products, but what % of revenue?

**Resolution Strategy:**
```python
# Strategy 1: Impact assessment
products_missing_category = products[products['product_category_name'].isnull()]
# Join with order_items to calculate revenue impact
revenue_impact = order_items.merge(products_missing_category, on='product_id')
# If <1% of revenue, can exclude or create "Unknown" category

# Strategy 2: Create "Uncategorized" category
products['product_category_name'].fillna('uncategorized', inplace=True)

# Strategy 3: For dimension attributes, consider:
# - Median imputation by category
# - Exclude from weight/size analysis
# - Flag for manual review

# Strategy 4: Check if English translation exists
missing_translation = set(products['product_category_name']) - set(category_translation['product_category_name'])
```

---

#### Reviews Table
**Issue:**
- `review_comment_title`: 87,656 missing (88.3%)
- `review_comment_message`: 58,247 missing (58.7%)

**Context:**
- This is expected user behavior (optional comment fields)
- Not missing data, but intentionally left blank
- Focus on `review_score` which has no missing values

**Resolution Strategy:**
```python
# No imputation needed - missing comments are valid
# For text analysis:
# - Analyze only reviews with comments
# - Calculate % with comments as engagement metric
# - Compare avg score for reviews with/without comments

engagement_rate = (reviews['review_comment_message'].notnull().sum() / len(reviews)) * 100
# Hypothesis: Users who leave comments may have stronger opinions (very happy or very unhappy)
```

---

### 2. Duplicate Records

#### Geolocation Table
**Issue:** 261,831 duplicate rows (26.2% of 1,000,163 records)

**Investigation:**
```python
# Check what defines a duplicate
geolocation.duplicated(subset=['geolocation_zip_code_prefix']).sum()
# vs
geolocation.duplicated().sum()

# Hypothesis: Same zip code has multiple lat/lng coordinates
# This could represent:
# 1. Geographic area of the zip code (multiple points)
# 2. Data quality issues (incorrect geocoding)
# 3. Multiple addresses within same zip
```

**Resolution Strategy:**
```python
# Strategy 1: Keep all records (represents geographic spread)
# Use case: Calculate average distance between customer and seller

# Strategy 2: Aggregate to centroid per zip code
geo_centroids = geolocation.groupby('geolocation_zip_code_prefix').agg({
    'geolocation_lat': 'mean',
    'geolocation_lng': 'mean',
    'geolocation_city': 'first',  # or most frequent
    'geolocation_state': 'first'
}).reset_index()

# Strategy 3: Keep unique zip-city-state combinations
geo_unique = geolocation.drop_duplicates(
    subset=['geolocation_zip_code_prefix', 'geolocation_city', 'geolocation_state']
)

# Recommendation: Use Strategy 2 (centroid) for distance calculations
```

---

### 3. Data Type Conversions

**Issue:** All timestamp columns stored as `object` instead of `datetime64`

**Columns to Convert:**
- `order_purchase_timestamp`
- `order_approved_at`
- `order_delivered_carrier_date`
- `order_delivered_customer_date`
- `order_estimated_delivery_date`
- `shipping_limit_date`
- `review_creation_date`
- `review_answer_timestamp`

**Resolution:**
```python
# Conversion function
def convert_timestamps(df, columns):
    for col in columns:
        if col in df.columns:
            df[col] = pd.to_datetime(df[col], errors='coerce')
            # errors='coerce' converts invalid parsing to NaT
    return df

# Apply to each dataframe
timestamp_cols_orders = [
    'order_purchase_timestamp',
    'order_approved_at',
    'order_delivered_carrier_date',
    'order_delivered_customer_date',
    'order_estimated_delivery_date'
]
orders = convert_timestamps(orders, timestamp_cols_orders)

# Verify conversion
orders.dtypes
orders['order_purchase_timestamp'].describe()  # Should show datetime statistics
```

---

### 4. Outlier Detection & Handling

#### Price and Freight Analysis
```python
# Identify outliers using IQR method
def detect_outliers_iqr(df, column):
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    
    outliers = df[(df[column] < lower_bound) | (df[column] > upper_bound)]
    return outliers, lower_bound, upper_bound

# Check price outliers
price_outliers, price_lower, price_upper = detect_outliers_iqr(order_items, 'price')
print(f"Price outliers: {len(price_outliers)} ({len(price_outliers)/len(order_items)*100:.2f}%)")
print(f"Price range: {price_lower:.2f} to {price_upper:.2f}")

# Check freight outliers
freight_outliers, freight_lower, freight_upper = detect_outliers_iqr(order_items, 'freight_value')

# Strategy: Don't remove outliers, but flag them for investigation
# Luxury items may legitimately cost 10x median price
```

#### Delivery Time Analysis
```python
# Calculate delivery time
orders['delivery_time_days'] = (
    orders['order_delivered_customer_date'] - 
    orders['order_purchase_timestamp']
).dt.total_seconds() / (24 * 3600)

# Check for impossible values
negative_delivery = orders[orders['delivery_time_days'] < 0]
extreme_delivery = orders[orders['delivery_time_days'] > 200]  # >6 months

# Investigation needed for:
# - Negative delivery times (data entry errors)
# - Extremely long delivery (>60 days)

# Resolution: Cap or exclude from specific analyses
orders_clean = orders[
    (orders['delivery_time_days'] >= 0) & 
    (orders['delivery_time_days'] <= 200)
]
```

---

### 5. Join Strategy & Data Integration

#### Primary Master Table Creation

**Objective:** Create analysis-ready master table with one row per order

```python
# Step 1: Start with orders as base
master = orders.copy()

# Step 2: Add customer information (1:1 join)
master = master.merge(
    customers,
    on='customer_id',
    how='left',
    validate='1:1'
)

# Step 3: Aggregate order items (Many:1 join)
order_summary = order_items.groupby('order_id').agg({
    'order_item_id': 'count',  # number of items
    'price': 'sum',  # total product value
    'freight_value': 'sum',  # total shipping cost
    'product_id': lambda x: list(x),  # list of products (optional)
    'seller_id': lambda x: list(x)  # list of sellers (optional)
}).reset_index()

order_summary.columns = [
    'order_id', 'total_items', 'total_price', 
    'total_freight', 'product_ids', 'seller_ids'
]

master = master.merge(
    order_summary,
    on='order_id',
    how='left',
    validate='1:1'
)

# Step 4: Aggregate payments (Many:1 join)
payment_summary = payments.groupby('order_id').agg({
    'payment_value': 'sum',
    'payment_type': lambda x: ', '.join(x.unique()),
    'payment_installments': 'max'
}).reset_index()

payment_summary.columns = [
    'order_id', 'total_payment_value', 
    'payment_methods', 'max_installments'
]

master = master.merge(
    payment_summary,
    on='order_id',
    how='left',
    validate='1:1'
)

# Step 5: Add reviews (1:0..1 join)
reviews_subset = order_reviews[[
    'order_id', 'review_score', 
    'review_comment_title', 'review_comment_message'
]]

master = master.merge(
    reviews_subset,
    on='order_id',
    how='left',
    validate='1:1'
)

# Step 6: Add customer geolocation (for distance calculations)
customer_geo = geolocation.groupby('geolocation_zip_code_prefix').agg({
    'geolocation_lat': 'mean',
    'geolocation_lng': 'mean'
}).reset_index()

customer_geo.columns = [
    'customer_zip_code_prefix', 'customer_lat', 'customer_lng'
]

master = master.merge(
    customer_geo,
    on='customer_zip_code_prefix',
    how='left',
    validate='m:1'
)

# Step 7: Validate master table
print(f"Master table shape: {master.shape}")
print(f"Expected rows: {len(orders)}")
print(f"Actual rows: {len(master)}")
assert len(master) == len(orders), "Row count mismatch in master table!"
```

#### Secondary Analysis Tables

**Product-Level Analysis:**
```python
# For category-level analysis
product_orders = order_items.merge(
    products,
    on='product_id',
    how='left'
).merge(
    category_translation,
    on='product_category_name',
    how='left'
).merge(
    orders[['order_id', 'order_delivered_customer_date', 
            'order_estimated_delivery_date', 'customer_state']],
    on='order_id',
    how='left'
).merge(
    order_reviews[['order_id', 'review_score']],
    on='order_id',
    how='left'
)
```

**Seller-Level Analysis:**
```python
# For seller performance analysis
seller_orders = order_items.merge(
    sellers,
    on='seller_id',
    how='left'
).merge(
    orders[['order_id', 'order_delivered_customer_date', 
            'order_estimated_delivery_date', 'customer_state']],
    on='order_id',
    how='left'
).merge(
    order_reviews[['order_id', 'review_score']],
    on='order_id',
    how='left'
)
```

---

### 6. Feature Engineering

#### Calculated Metrics

```python
# Delivery performance metrics
master['delivery_time_days'] = (
    master['order_delivered_customer_date'] - 
    master['order_purchase_timestamp']
).dt.total_seconds() / (24 * 3600)

master['estimated_delivery_time_days'] = (
    master['order_estimated_delivery_date'] - 
    master['order_purchase_timestamp']
).dt.total_seconds() / (24 * 3600)

master['delivery_delay_days'] = (
    master['order_delivered_customer_date'] - 
    master['order_estimated_delivery_date']
).dt.total_seconds() / (24 * 3600)

# Delivery status categories
def categorize_delivery(delay_days):
    if pd.isna(delay_days):
        return 'Not Delivered'
    elif delay_days < -2:
        return 'Very Early (>2 days)'
    elif delay_days < 0:
        return 'Early'
    elif delay_days == 0:
        return 'On Time'
    elif delay_days <= 3:
        return 'Slightly Late (1-3 days)'
    elif delay_days <= 7:
        return 'Late (4-7 days)'
    else:
        return 'Very Late (>7 days)'

master['delivery_status'] = master['delivery_delay_days'].apply(categorize_delivery)

# On-time delivery flag
master['is_on_time'] = master['delivery_delay_days'] <= 0
master['is_late'] = master['delivery_delay_days'] > 0

# Order value metrics
master['order_total'] = master['total_price'] + master['total_freight']
master['avg_item_price'] = master['total_price'] / master['total_items']
master['freight_percentage'] = (master['total_freight'] / master['order_total']) * 100

# Temporal features
master['purchase_year'] = master['order_purchase_timestamp'].dt.year
master['purchase_month'] = master['order_purchase_timestamp'].dt.month
master['purchase_quarter'] = master['order_purchase_timestamp'].dt.quarter
master['purchase_day_of_week'] = master['order_purchase_timestamp'].dt.dayofweek
master['purchase_hour'] = master['order_purchase_timestamp'].dt.hour
master['purchase_month_name'] = master['order_purchase_timestamp'].dt.month_name()

# Review metrics
master['has_review'] = master['review_score'].notnull()
master['has_comment'] = master['review_comment_message'].notnull()
master['is_positive_review'] = master['review_score'] >= 4
master['is_negative_review'] = master['review_score'] <= 2

# Payment metrics
master['uses_installments'] = master['max_installments'] > 1
master['installment_value'] = master['total_payment_value'] / master['max_installments']

# Order complexity
master['is_multi_item'] = master['total_items'] > 1
master['order_size_category'] = pd.cut(
    master['total_items'],
    bins=[0, 1, 2, 5, float('inf')],
    labels=['Single Item', '2 Items', '3-5 Items', '6+ Items']
)
```

#### Distance Calculation (Advanced)

```python
from math import radians, cos, sin, asin, sqrt

def haversine(lon1, lat1, lon2, lat2):
    """
    Calculate distance between two points on earth in kilometers
    """
    lon1, lat1, lon2, lat2 = map(radians, [lon1, lat1, lon2, lat2])
    dlon = lon2 - lon1 
    dlat = lat2 - lat1 
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    c = 2 * asin(sqrt(a)) 
    r = 6371  # Earth radius in kilometers
    return c * r

# Apply to calculate customer-seller distance
# (Requires joining seller geolocation data)
```

---

## 📈 Power BI Dashboard Design

### Data Modeling in Power BI

#### Relationships
```
customers (1) ←→ (*) orders
orders (1) ←→ (*) order_items
order_items (*) ←→ (1) products
order_items (*) ←→ (1) sellers
orders (1) ←→ (*) payments
orders (1) ←→ (1) reviews
products (*) ←→ (1) category_translation
```

#### DAX Measures - Core Calculations

```dax
// Revenue Metrics
Total Revenue = SUM(payments[payment_value])

Total Orders = DISTINCTCOUNT(orders[order_id])

Average Order Value = DIVIDE([Total Revenue], [Total Orders])

Revenue Growth MoM = 
VAR CurrentRevenue = [Total Revenue]
VAR PreviousRevenue = CALCULATE([Total Revenue], DATEADD('Date'[Date], -1, MONTH))
RETURN DIVIDE(CurrentRevenue - PreviousRevenue, PreviousRevenue)

// Delivery Metrics
Average Delivery Time = 
AVERAGE(
    DATEDIFF(
        orders[order_purchase_timestamp],
        orders[order_delivered_customer_date],
        DAY
    )
)

On-Time Delivery Rate = 
VAR OnTimeOrders = 
    CALCULATE(
        COUNTROWS(orders),
        orders[order_delivered_customer_date] <= orders[order_estimated_delivery_date]
    )
VAR TotalDelivered = 
    CALCULATE(
        COUNTROWS(orders),
        NOT(ISBLANK(orders[order_delivered_customer_date]))
    )
RETURN DIVIDE(OnTimeOrders, TotalDelivered)

Late Delivery Rate = 1 - [On-Time Delivery Rate]

Average Delay Days = 
AVERAGE(
    DATEDIFF(
        orders[order_estimated_delivery_date],
        orders[order_delivered_customer_date],
        DAY
    )
)

// Review Metrics
Average Review Score = AVERAGE(reviews[review_score])

Positive Review Rate = 
DIVIDE(
    CALCULATE(COUNTROWS(reviews), reviews[review_score] >= 4),
    COUNTROWS(reviews)
)

Review Response Rate = 
DIVIDE(
    COUNTROWS(reviews),
    DISTINCTCOUNT(orders[order_id])
)

// Customer Metrics
Total Customers = DISTINCTCOUNT(customers[customer_unique_id])

Repeat Customer Rate = 
VAR RepeatCustomers = 
    CALCULATE(
        DISTINCTCOUNT(customers[customer_unique_id]),
        FILTER(
            VALUES(customers[customer_unique_id]),
            CALCULATE(DISTINCTCOUNT(orders[order_id])) > 1
        )
    )
RETURN DIVIDE(RepeatCustomers, [Total Customers])

// Seller Metrics
Total Sellers = DISTINCTCOUNT(sellers[seller_id])

Seller Efficiency Score = 
[On-Time Delivery Rate] * 0.5 + 
([Average Review Score] / 5) * 0.5
```

---

### Page 1: Executive Overview

#### Layout Structure & Components

**KPI Cards (Top Row):**
1. **Total Orders**
   - Value: 99,441
   - Trend indicator: MoM growth %
   - Conditional formatting: Green if growth > 0

2. **Total Revenue** 
   - Value: Sum of payment_value
   - Format: Currency (R$)
   - Comparison to previous period

3. **Average Delivery Time**
   - Value: Average days from purchase to delivery
   - Benchmark: Industry standard or target
   - Color: Red if > 15 days, Yellow if 10-15, Green if < 10

4. **On-Time Delivery Rate**
   - Value: % of orders delivered by estimated date
   - Target line at 95%
   - Historical trend sparkline

5. **Average Review Score**
   - Value: 1-5 star rating
   - Visual: Star icons
   - Comparison to target (4.0+)

**Monthly Trend Chart:**
- **Type:** Dual-axis line chart
- **Primary Axis:** Order count (bars)
- **Secondary Axis:** Revenue (line)
- **X-Axis:** Month-Year
- **Features:**
  - Tooltips showing exact values
  - Trend lines for both metrics
  - Highlight peak months
  - Date slicer for filtering

**Key Insights Box:**
- **Type:** Text box / Card visual
- **Content Format:**
  ```
  📊 KEY INSIGHTS (Updated: [Date])
  
  ✓ 92.3% on-time delivery rate (↑2.1% vs Q3)
  ⚠ SP state shows 15% higher delays than average
  ✓ Credit card orders have 8% higher completion rate
  ⚠ Electronics category has lowest review scores (3.6)
  ✓ Revenue grew 12% MoM, driven by increased AOV
  ```

**Filters/Slicers:**
- Date range (month/quarter/year)
- State dropdown
- Order status multi-select
- Product category

**Interactivity:**
- Click on month → filters all other pages
- Click on KPI → drills to relevant detail page
- Hover tooltips showing detailed metrics