# Customer Shopping Behavior Analysis (Python + SQL)

An end-to-end analysis of retail customer shopping data. The project uses **Python (pandas)** to clean and prepare the data, loads it into a relational database (**PostgreSQL**, with MySQL and SQL Server alternatives), and uses **SQL** to answer ten business questions about revenue, discounts, subscriptions, loyalty, and product performance.

## Tools Used

- **Python:** pandas, SQLAlchemy, psycopg2 / pymysql / pyodbc
- **SQL:** PostgreSQL (CTEs, window functions, subqueries, conditional aggregation)
- **Jupyter Notebook**

## Business Problem

A retail company wants to understand its customers' shopping behavior in order to improve sales, customer satisfaction, and long-term loyalty. Management has noticed shifting purchase patterns across demographics, product categories, and sales channels, and wants to know which factors (discounts, reviews, seasons, payment preferences) drive purchase decisions and repeat buying.

> **Core question:** *How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?*

The full problem statement is in [`Business Problem Statement.pdf`](Business%20Problem%20Statement.pdf).

## Repository Contents

| File | Description |
|------|-------------|
| `Business Problem Statement.pdf` | The business context and the overarching question |
| `customer_shopping_behavior.csv` | Raw dataset (3,900 rows, 18 columns) |
| `Customer_Shopping_Behavior_Analysis.ipynb` | Jupyter notebook: data loading, cleaning, feature engineering, and database export |
| `customer_behavior_sql_queries.sql` | 10 SQL queries answering the business questions |

## Dataset

3,900 customer transactions with 18 columns:

| Column | Description |
|--------|-------------|
| `Customer ID` | Unique customer identifier |
| `Age`, `Gender` | Customer demographics (ages 18-70) |
| `Item Purchased`, `Category` | Product (25 unique items) and its category (4 categories) |
| `Purchase Amount (USD)` | Transaction value |
| `Location` | US state (50 unique) |
| `Size`, `Color`, `Season` | Product attributes and season of purchase |
| `Review Rating` | Customer rating of the product |
| `Subscription Status` | Whether the customer is subscribed |
| `Shipping Type` | Shipping method (e.g. Standard, Express, Free Shipping, Next Day Air) |
| `Discount Applied`, `Promo Code Used` | Whether a discount or promo code was used |
| `Previous Purchases` | Number of earlier purchases by the customer |
| `Payment Method` | e.g. Credit Card, PayPal, Venmo, Cash |
| `Frequency of Purchases` | How often the customer buys (Weekly, Fortnightly, Monthly, Quarterly, Annually, etc.) |

## Workflow

### 1. Data preparation in Python (`.ipynb`)

- **Load and inspect** the CSV with `df.head()`, `df.info()`, and `df.describe(include='all')`.
- **Handle missing values:** `Review Rating` had 37 nulls, imputed with the **median rating of the product category**.
- **Standardize column names** to snake_case (e.g. `Purchase Amount (USD)` becomes `purchase_amount`).
- **Feature engineering:**
  - `age_group`: customers split into four quartile-based groups (Young Adult, Adult, Middle-aged, Senior) using `pd.qcut`.
  - `purchase_frequency_days`: converts the text frequency (Weekly, Monthly, Quarterly, ...) into an approximate number of days between purchases.
- **Drop redundancy:** `promo_code_used` was verified to be identical to `discount_applied` on every row, so it was removed.

### 2. Load into a database

The cleaned DataFrame is written to a table named `customer` in a database named `customer_behavior` using SQLAlchemy. The notebook includes connection code for:

- **PostgreSQL** (`psycopg2`), which is the one the SQL queries were written for
- **MySQL** (`pymysql`)
- **Microsoft SQL Server** (`pyodbc`)

### 3. Business analysis in SQL

`customer_behavior_sql_queries.sql` answers ten questions:

| # | Business question | Techniques |
|---|-------------------|-----------|
| 1 | Total revenue from male vs. female customers | `GROUP BY`, `SUM` |
| 2 | Customers who used a discount but still spent above the average purchase amount | Subquery |
| 3 | Top 5 products by average review rating | `AVG`, `ORDER BY`, `LIMIT` |
| 4 | Average purchase amount: Standard vs. Express shipping | `AVG`, filtering |
| 5 | Do subscribers spend more? Compare average spend and total revenue | Multi-aggregate `GROUP BY` |
| 6 | Top 5 products by percentage of purchases made with a discount | Conditional aggregation (`CASE`) |
| 7 | Segment customers into New, Returning, and Loyal and count each | CTE, `CASE` |
| 8 | Top 3 most purchased products within each category | CTE, `ROW_NUMBER() OVER (PARTITION BY ...)` |
| 9 | Are repeat buyers (more than 5 previous purchases) likely to subscribe? | Filtering, `GROUP BY` |
| 10 | Revenue contribution of each age group | `GROUP BY`, `SUM` |

## Getting Started

### Prerequisites

- Python 3.8+
- Jupyter Notebook or JupyterLab
- A SQL database (PostgreSQL recommended) with an empty database named `customer_behavior`

```bash
pip install pandas sqlalchemy psycopg2-binary jupyter
# For MySQL:      pip install pymysql
# For SQL Server: pip install pyodbc   (also requires Microsoft ODBC Driver 17)
```

### Run the project

1. Clone the repository and place the files in one folder.
2. Create the database, e.g. in PostgreSQL:
   ```sql
   CREATE DATABASE customer_behavior;
   ```
3. Open `Customer_Shopping_Behavior_Analysis.ipynb` and run the cells in order. In the connection cell, **replace the username, password, host, and port with your own credentials.**
4. Once the `customer` table is created, open `customer_behavior_sql_queries.sql` in pgAdmin, DBeaver, or `psql` and run each query.

### Dialect notes

The queries target PostgreSQL. If you use another database, small changes may be needed:

- Q3 uses `review_rating::numeric` (PostgreSQL cast syntax). Use `CAST(review_rating AS DECIMAL(10,2))` elsewhere.
- Q3, Q6 use `LIMIT`. SQL Server uses `TOP 5` instead.
