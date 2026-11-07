# Customer Shopping Behavior Analysis

An end-to-end analysis of 3,900 retail purchases: cleaning and feature engineering in **Python**, loading into **PostgreSQL**, and answering ten business questions with **SQL**.

![Python](https://img.shields.io/badge/Python-3.12-3670A0?style=flat-square&logo=python&logoColor=ffdd54)
![pandas](https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat-square)

## Business question

A retail company has noticed shifting purchase patterns across demographics, product categories and channels. Management wants to know which factors (discounts, reviews, seasons, subscriptions, shipping) drive spending and repeat purchases:

> *How can the company use its shopping data to identify trends, improve customer engagement and optimize marketing and product strategy?*

The full brief is in [`Business-Problem.md`](Business-Problem.md).

## Pipeline

```
customer_shopping_behavior.csv
        │
        ▼
Python (pandas) ── clean, impute, engineer features
        │
        ▼
PostgreSQL ── table `customer` (via SQLAlchemy)
        │
        ▼
SQL ── 10 business questions
```

## Data preparation

The notebook [`Customer_Shoping_Behavior_Analysis.ipynb`](Customer_Shoping_Behavior_Analysis.ipynb) does the following:

| Step | What and why |
|---|---|
| Profile | 3,900 rows × 18 columns. Only `Review Rating` has missing values (37 rows). |
| Impute | Missing ratings are filled with the **median rating of the same product category**, so the fill respects category differences instead of using one global value. |
| Standardise columns | Lower-case `snake_case` names, e.g. `Purchase Amount (USD)` → `purchase_amount`, so SQL queries don't need quoted identifiers. |
| `age_group` | Ages (18–70) are split into four equal-sized quartiles: Young Adult, Adult, Middle-aged, Senior. |
| `purchase_frequency_days` | Text frequencies (`Weekly`, `Quarterly`, …) are mapped to an approximate number of days, so frequency can be treated as a number. |
| Drop redundant column | `promo_code_used` was identical to `discount_applied` in every row, so it was removed. |
| Load | The cleaned DataFrame is written to PostgreSQL as table `customer`. |

## Business questions answered in SQL

All queries are in [`Data/customer_behavior_sql_queries.sql`](Data/customer_behavior_sql_queries.sql).

| # | Question | Technique |
|---|---|---|
| Q1 | Revenue by gender | `GROUP BY` |
| Q2 | Customers who used a discount but still spent above average | Subquery |
| Q3 | Top 5 products by average review rating | Aggregation, `ORDER BY … LIMIT` |
| Q4 | Average spend: Standard vs. Express shipping | Filtered aggregation |
| Q5 | Do subscribers spend more? | Count, average and total by segment |
| Q6 | Top 5 products by share of discounted purchases | Conditional aggregation (`CASE` inside `SUM`) |
| Q7 | New / Returning / Loyal customer segments | CTE + `CASE` |
| Q8 | Top 3 products in each category | Window function (`ROW_NUMBER() OVER (PARTITION BY …)`) |
| Q9 | Are repeat buyers more likely to subscribe? | Filtered grouping |
| Q10 | Revenue contribution by age group | Aggregation on an engineered feature |

## Key findings

Total revenue in the dataset is **$233,081** across 3,900 purchases, an average of **$59.76** per purchase.

1. **Subscriptions are not linked to higher spending.** Subscribers spend $59.49 per purchase on average, compared with $59.87 for non-subscribers. Among repeat buyers (more than 5 previous purchases), 958 are subscribed and 2,518 are not. The subscription programme is not capturing the most loyal customers, which makes it the clearest opportunity for the marketing team.
2. **Discounts don't stop customers from spending.** 43% of purchases used a discount, and 839 of those discounted purchases were still at or above the average purchase amount. Hats (50%), sneakers and coats (about 49% each) are the most discount-dependent products.
3. **Clothing drives revenue.** Clothing brings in $104,264 (45% of revenue), followed by accessories ($74,200), footwear ($36,093) and outerwear ($18,524).
4. **Revenue comes mostly from male customers because there are more of them.** Male customers account for $157,890 compared with $75,191 for female customers, but the average purchase is almost the same ($59.54 vs. $60.25). This is a reach difference, not a spending difference.
5. **Express shipping comes with slightly larger baskets:** $60.48 compared with $58.46 for Standard.
6. **Young adults contribute the most revenue** ($62,143), but the gap to the other age groups is small, between $55,763 and $59,197.
7. **The loyalty thresholds need rethinking.** With "Loyal" defined as more than 10 previous purchases, 80% of customers (3,116) fall into that segment and only 83 are "New". A segmentation where most customers share one label can't guide targeting. Quantile-based thresholds would give more useful groups.

> **About the data:** many differences in this dataset are small, such as rating averages between 3.78 and 3.86 and per-purchase averages within about $1. Treat findings 4–6 as directions to investigate, not as proven effects.

## How to run

1. Create the environment:

   ```bash
   conda env create -f Environtment.yml
   conda activate dtenv
   ```

2. Create an empty PostgreSQL database named `customer_behavior`.

3. Set your database password as an environment variable. The notebook reads it from there, so it never lives in the code:

   ```bash
   # Windows (PowerShell)
   $env:PG_PASSWORD = "your-password"
   # macOS / Linux
   export PG_PASSWORD="your-password"
   ```

4. Run the notebook from top to bottom. It loads the cleaned data into the `customer` table.

5. Run the queries in `Data/customer_behavior_sql_queries.sql` with pgAdmin, DBeaver or `psql`.

## Repository structure

```
customer_behavior_analysis/
├── Business-Problem.md                     # project brief
├── Customer_Shoping_Behavior_Analysis.ipynb # cleaning, features, load to PostgreSQL
├── Data/
│   ├── customer_shopping_behavior.csv      # raw dataset (3,900 rows)
│   └── customer_behavior_sql_queries.sql   # 10 business questions
├── Environtment.yml                        # conda environment
├── requirements.txt                        # pip alternative
└── LICENSE
```

## Next steps

- A Power BI dashboard covering revenue by category, subscription versus repeat-buyer behaviour, and discount dependency per product
- A data-driven loyalty segmentation (for example RFM scoring) to replace the fixed thresholds
- Statistical tests for the small differences before turning them into recommendations

## License

[MIT](LICENSE) © 2025 Rijal Farizky
