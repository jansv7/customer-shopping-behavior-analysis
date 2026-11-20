# Customer Shopping Behavior Analysis

An end-to-end data analysis project that turns raw retail transaction data into business insights. The workflow covers data acquisition and cleaning in **Python**, structured business analysis in **SQL**, an interactive **Power BI** dashboard, and a written report + presentation with actionable recommendations.

---

## Table of Contents

- [Business Problem](#business-problem)
- [Project Objectives](#project-objectives)
- [Dataset](#dataset)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Methodology](#methodology)
  - [1. Data Preparation & Modeling (Python)](#1-data-preparation--modeling-python)
  - [2. Data Analysis (SQL)](#2-data-analysis-sql)
  - [3. Visualization (Power BI)](#3-visualization-power-bi)
  - [4. Reporting & Presentation](#4-reporting--presentation)
- [Key Findings](#key-findings)
- [Business Recommendations](#business-recommendations)
- [How to Run](#how-to-run)
- [Deliverables](#deliverables)
- [Author](#author)

---

## Business Problem

A leading retail company wants to better understand its customers' shopping behavior in order to improve sales, customer satisfaction, and long-term loyalty. Management has noticed shifts in purchasing patterns across demographics, product categories, and sales channels, and wants to know which factors — discounts, reviews, seasons, or payment preferences — actually drive consumer decisions and repeat purchases.

**Overarching business question:**

> How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?

## Project Objectives

1. Clean and transform the raw transactional dataset into an analysis-ready form.
2. Structure the data in a relational database and answer business questions with SQL.
3. Build an interactive dashboard that surfaces the key patterns and trends for stakeholders.
4. Summarize findings and translate them into concrete business recommendations.

## Dataset

| Attribute | Detail |
|---|---|
| File | `Dataset/customer_shopping_behavior.csv` |
| Rows | 3,900 |
| Columns | 18 |
| Missing values | 37 (all in `Review Rating`) |

**Feature groups**

- **Demographics** — `Customer ID`, `Age`, `Gender`, `Location`, `Subscription Status`
- **Purchase details** — `Item Purchased`, `Category`, `Purchase Amount (USD)`, `Season`, `Size`, `Color`
- **Shopping behavior** — `Discount Applied`, `Promo Code Used`, `Previous Purchases`, `Frequency of Purchases`, `Review Rating`, `Shipping Type`, `Payment Method`

**Quick stats:** average purchase amount **$59.76**, median **$60**, range $20–$100 · customer age 18–70 (median 44) · 4 categories, 25 products, 50 locations.

## Tech Stack

| Layer | Tools |
|---|---|
| Data preparation | Python 3.11, pandas, NumPy |
| Data modeling / EDA | Jupyter Notebook, pandas |
| Database | MySQL / MariaDB via SQLAlchemy + PyMySQL |
| Business analysis | SQL (CTEs, window functions, aggregations) |
| Visualization | Microsoft Power BI Desktop |
| Reporting | Microsoft Word / PDF, PowerPoint |

## Repository Structure

```
project_data_analysis/
├── README.md
├── Data Analysis.ipynb                      # EDA, cleaning, feature engineering, DB load
├── Dataset/
│   └── customer_shopping_behavior.csv       # Raw dataset (3,900 rows × 18 columns)
├── Bussiness Problem/
│   └── Bussiness Problem Statement.pdf      # Task brief and business question
├── Bussiness Analysis Using SQL/
│   └── sql_query_analysis.sql               # 10 business queries (Q1–Q10)
├── Power BI Visualization/
│   └── dashboard_behavior_analysis.pbix     # Interactive dashboard
├── Analysis Result/
│   ├── Customer Shopping Behavior Analysis.docx
│   └── Customer Shopping Behavior Analysis.pdf
└── PPT/
    └── CustomerShoppingBehaviorAnalysis.pptx
```

## Methodology

### 1. Data Preparation & Modeling (Python)

`Data Analysis.ipynb`

**Data Acquisition** — loaded `customer_shopping_behavior.csv` into a pandas DataFrame.

**Data Understanding** — inspected structure with `df.info()` and summary statistics with `df.describe(include='all')`; `df.isna().sum()` isolated 37 missing values in `Review Rating`.

**Data Preprocessing**
- Imputed the 37 missing `Review Rating` values using the **median rating of each product category** (keeps the fill consistent within a category rather than using one global average).
- Standardized column names to `snake_case` for readability and easier SQL mapping.

**Feature Engineering**
- `age_group` — customers binned into quartiles: **Young Adult, Adult, Middle-aged, Senior**.
- `purchase_frequency_days` — mapped the text purchase frequency (`Weekly` → 7, `Fortnightly`/`Bi-Weekly` → 14, `Monthly` → 30, `Quarterly`/`Every 3 Months` → 90, `Annually` → 365) into a numeric interval.
- **Redundancy check** — verified `discount_applied` and `promo_code_used` were identical in every row, then dropped `promo_code_used`.

**Database Integration** — the cleaned DataFrame (19 columns) was loaded into the `customer` table of the `customer_behavior` database via SQLAlchemy for the SQL stage.

### 2. Data Analysis (SQL)

`Bussiness Analysis Using SQL/sql_query_analysis.sql` — 10 business questions answered over the `customer` table:

| # | Question | Techniques |
|---|---|---|
| Q1 | Total revenue by gender | `SUM`, `GROUP BY` |
| Q2 | Customers who used a discount yet still spent above the overall average | Subquery, `WHERE` |
| Q3 | Top 5 products by average review rating | `AVG`, `ORDER BY`, `LIMIT` |
| Q4 | Average purchase amount: Standard vs Express shipping | Filtered `GROUP BY` |
| Q5 | Subscribers vs non-subscribers: customer count, average spend, total revenue | Multi-metric aggregation |
| Q6 | Top 5 products with the highest share of discounted purchases | `CASE WHEN` ratio |
| Q7 | Customer segmentation into New / Returning / Loyal | CTE, `CASE` |
| Q8 | Top 3 most purchased products within each category | CTE, `ROW_NUMBER() OVER (PARTITION BY …)` |
| Q9 | Are repeat buyers (>5 previous purchases) more likely to subscribe? | Filtered `GROUP BY` |
| Q10 | Revenue contribution of each age group | `SUM`, `GROUP BY` |

### 3. Visualization (Power BI)

`Power BI Visualization/dashboard_behavior_analysis.pbix` — a single interactive dashboard summarizing revenue by demographic, product/category performance, discount dependency, subscription split, and shipping behavior, so stakeholders can filter and drill into the patterns themselves.

### 4. Reporting & Presentation

- **Report** — `Analysis Result/Customer Shopping Behavior Analysis.docx` / `.pdf`: project overview, dataset summary, methodology, findings, and recommendations.
- **Deck** — `PPT/CustomerShoppingBehaviorAnalysis.pptx`: a stakeholder-facing walkthrough from problem statement to recommendations.

## Key Findings

- **Revenue is heavily male-skewed in this dataset** — Male customers: 2,652 orders / **$157,890**; Female customers: 1,248 orders / **$75,191** (total revenue $233,081).
- **Subscriptions barely change basket size.** Subscribers average **$59.49** per purchase (1,053 customers, $62,645) vs non-subscribers at **$59.87** (2,847 customers, $170,436) — a negligible gap, so the value of subscribing is not showing up in order value.
- **Discounts don't attract bigger baskets either.** Discount users average **$59.28** vs **$60.13** without — yet 839 customers used a discount *and* still spent above the average, meaning the discount was not the reason for the spend.
- **Discount dependency is concentrated in a handful of products** — Hat and Sneakers (50.0% of purchases discounted), Coat (49.0%), Sweater (48.0%), Boots (47.0%). These are the items where margin is leaking.
- **Top-rated products:** Gloves (3.86), Sandals (3.84), Boots (3.82), followed by Hat (3.80) and Skirt (3.79).
- **Younger customers generate the most revenue** — Young Adult $62,143, Middle-aged $59,197, Adult $55,978, Senior $55,763.
- **Express shipping baskets are slightly larger** — $60.48 vs $58.46 for Standard.
- **Repeat buyers are not subscribing** — of customers with more than 5 previous purchases, 2,518 are non-subscribers vs only 958 subscribers.

## Business Recommendations

1. **Fix the subscription value proposition** — since subscriber spend is statistically flat versus non-subscribers, offer exclusive benefits (free shipping, early access, member-only pricing) rather than relying on the program to grow baskets by itself.
2. **Build loyalty programs that target repeat buyers** — 2,518 high-frequency non-subscribers are the most convertible audience; reward them to move them into the “Loyal” segment and into subscriptions.
3. **Review the discount policy** — discounts are not lifting average order value, so shift from blanket discounts to targeted, margin-aware campaigns focused on the five discount-dependent products.
4. **Product positioning** — push top-rated items (Gloves, Sandals, Boots) and best-sellers in campaigns instead of discounting everything.
5. **Targeted marketing** — prioritize high-revenue age groups (Young Adult, Middle-aged) and express-shipping customers, and lean into the male-dominated revenue base while testing campaigns to grow the female segment.

## How to Run

**Prerequisites:** Python 3.11+, MySQL/MariaDB, Power BI Desktop (for the dashboard), Jupyter.

```bash
# 1. Clone
git clone https://github.com/jansv7/customer-shopping-behavior-analysis.git
cd customer-shopping-behavior-analysis

# 2. Create and activate a virtual environment
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS / Linux

# 3. Install dependencies
pip install pandas numpy jupyter sqlalchemy pymysql

# 4. Launch the notebook
jupyter notebook "Data Analysis.ipynb"
```

**Database setup before running the DB cell:** create the schema once —

```sql
CREATE DATABASE customer_behavior;
```

then update the connection block in the notebook if your credentials differ, and re-run it. It writes the cleaned DataFrame to the `customer` table (`if_exists="replace"`):

```python
username = "root"
password = ""
host     = "localhost"
port     = "3306"
database = "customer_behavior"
```

Run the queries in `Bussiness Analysis Using SQL/sql_query_analysis.sql` against that database, then open the `.pbix` file in Power BI Desktop to explore the dashboard.

## Deliverables

| # | Deliverable | File |
|---|---|---|
| 1 | Data Preparation & Modeling (Python) | `Data Analysis.ipynb` |
| 2 | Data Analysis (SQL) | `Bussiness Analysis Using SQL/sql_query_analysis.sql` |
| 3 | Visualization & Insights (Power BI) | `Power BI Visualization/dashboard_behavior_analysis.pbix` |
| 4 | Report & Presentation | `Analysis Result/*.docx`, `Analysis Result/*.pdf`, `PPT/CustomerShoppingBehaviorAnalysis.pptx` |
| 5 | GitHub Repository | This repository |

## Author

**Kelompok 4** — Data Analysis project.
Repository: <https://github.com/jansv7/customer-shopping-behavior-analysis>
