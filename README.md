# Supply-Chain-Data-Cleaning-Relational-Modeling

---

## 1. Project Overview

This project focuses on transforming raw supply chain data into a clean, normalized, SQL-ready relational database suitable for analytical and strategic use. It represents the data preparation and modeling phase of a broader supply chain analytics workflow, ensuring that downstream analyses are built on reliable and well-structured data.
A single denormalized CSV file sourced from a Supply Chain Data Hub was cleaned, standardized, and decomposed into 8 relational tables using Python. The final output was loaded into PostgreSQL for structured querying and analysis.

### Objectives

To transform a raw, denormalized supply chain dataset into a clean, normalized, SQL-ready relational schema that:
- Preserves data integrity
- Eliminates redundancy through normalization
- Supports scalable querying and downstream analytics
- Serves as the authoritative data layer for warehouse location and capacity planning
- Enables efficient joins and queries for downstream analytics

---

## 2. Data Source and Initial Challenges
The source dataset was downloaded from [supply chain datahub](https://supplychaindatahub.org/) and consisted of:
- 155,488 rows
- 41 columns
- A single flat CSV file
Key challenges identified during initial assessment included:
- Presence of derived columns that inflated complexity and could be recomputed later
- Placeholder values in order and shipping date fields
- Redundant product and category records
- Inconsistent geographic attributes (country, state, and city values)
- A fully denormalized structure unsuitable for SQL-based analysis
These issues made the dataset inappropriate for direct analytical use without significant preprocessing.

---

## 3. Data Cleaning & Normalization Approach

### 1. Initial Data Assessment & Column Pruning

- Loaded the raw CSV into Python (pandas)
- Identified and dropped derived columns that could be reliably recreated during analysis (e.g., calculated metrics), reducing noise and improving schema clarity

### 2. Relational Table Design & Decomposition

For remaining attributes I decomposed into core transactional and dimension tables, aligned with relational database best practices:
Core Tables
- Customers
- Orders
- Order_Items
- Products
- Categories

```python
customers = df[['customer_id','customer_city','customer_state','customer_zipcode','customer_country','customer_segment']]
products = df[['product_card_id','product_name','category_id']]
categories = df[['category_id','category_name','department_name']]
orders = df[['order_id','customer_id','order_date','shipping_date','order_status','order_city','order_state','order_country','market','order_region','shipping_mode','payment_type']]
order_items = df[['order_id','product_card_id','product_price','order_item_quantity','order_item_discount','order_item_discount_rate']]
```

This step converted a single flat file into a logical relational structure, enabling joins, aggregations, and SQL-based analytics.
To ensure referential accuracy, Product and category tables were collapsed to their true cardinality using de-duplication
- 118 unique products
- 51 unique categories

```python
products = products.drop_duplicates()
categories = categories.drop_duplicates()
```

### 3. Date Integrity & Synthetic Generation

Order and shipping dates contained placeholders instead of valid timestamps.
I Generated realistic dates using seasonal weighting, preserving plausible order and shipping patterns while avoiding analytical bias
This ensured time-series analysis (lead times, seasonality, demand trends) would be valid in downstream SQL queries.

### 4. Dimensional Normalization for Data Integrity

To further reduce redundancy and enforce consistency, I created 3 additional dimension tables for better data intergrity.
- customer_segments
- payment_types
- shipping_mode
Each dimension was assigned surrogate keys and mapped back to the Orders table:

```python
unique_payments = orders['payment_type'].unique()
payment_mapping = {payment: i+1 for i, payment in enumerate(unique_payments)}
orders.loc[:,'payment_id'] = orders['payment_type'].map(payment_mapping)
orders = orders.drop(columns='payment_type')
```

### 5. Targeted Data Cleaning & Standardization
Focused cleaning was applied at the table level, with an emphasis on analytical correctness rather than cosmetic fixes:
- Standardized customer country values (e.g., converting “EE.UU” to “USA”)
- Corrected state-level inconsistencies where ZIP codes were incorrectly stored as state abbreviations (notably in California)
- Reassigned invalid city entries using controlled randomness from a curated list of major California cities to preserve geographic realism
This ensured geospatial analyses (regional demand, warehouse coverage, shipping zones) would be credible.

```python
customers.loc[customers['customer_country'] == 'EE. UU.', 'customer_country'] = 'United States'
customers.loc[customers['customer_state'] == '95758', 'customer_state'] = 'CA'
```

```python
mask = (customers['customer_city'] == 'CA') & (customers['customer_state'] == 'CA')
top_ca_cities = ["Los Angeles","San Diego","San Jose","San Francisco","Fresno","Sacramento","Long Beach","Oakland"]
customers.loc[mask, 'customer_city'] = np.random.choice(top_ca_cities, size= mask.sum())
customers.loc[customers['customer_city'] == 'CA']
```

6. Export & SQL Integration
After validation:
- All 8 finalized tables were exported as clean CSV files.
- Loaded into PostgreSQL, forming the structured data layer used in the subsequent warehouse expansion and logistics optimization analysis.

```python
customers.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\customers.csv', index=False)
products.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\products.csv', index=False)
categories.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\categories.csv', index=False)
orders.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\orders.csv', index=False)
order_items.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\order_items.csv', index=False)
customer_segments.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\customer_segments.csv', index=False)
payment_types.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\payment_types.csv', index=False)
shipping_mode.to_csv(r'C:\Users\user\Desktop\LAST_FOLDER\Postgres_files\shipping_mode.csv', index=False)
```

---

## Final Schema (8 Tables)

- customers
- products
- categories
- orders
- orderitems
- customer_segments
- payment_types
- shipping mode

---
## Final Deliverables

- 8 normalized, SQL-ready relational tables.
- Cleaned and validated customer, product, order, and logistics dimensions.
- A production-style schema suitable for joins, aggregations, and strategic modeling.

--- 

## Tools & Skills Demonstrated
- Python (pandas, NumPy)
- Data cleaning and validation
- Relational data modeling
- Dimensional normalization
- PostgreSQL integration
- Analytical judgment in handling imperfect real-world data

--- 


