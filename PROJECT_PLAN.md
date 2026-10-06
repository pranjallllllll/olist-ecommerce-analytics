# Olist E-Commerce Analytics Project

## Project Goal

Build an end-to-end business analytics project using real, anonymized
commercial e-commerce data from Olist.

The goal is to understand the business through its sales, customers,
products, sellers, orders, payments, deliveries, and reviews, and then
turn the findings into meaningful business insights and recommendations.

This project is intended to demonstrate practical skills in:

- Excel / Google Sheets
- SQL
- Python
- Pandas
- Statistics
- Power BI
- Machine Learning
- Git & GitHub

The focus is not just on creating charts or using tools, but on solving
business problems and explaining why each analysis is useful.

---

# Project Workflow

## Stage 1 — Data Exploration
Tool: Python + Pandas

Understand the raw datasets before making major changes.

Tasks:
- Inspect all available datasets
- Understand rows and columns
- Understand what each table represents
- Check data types
- Check missing values
- Check duplicates
- Explore unique values
- Understand date ranges
- Identify unusual or inconsistent values
- Understand relationships between tables

Output:
`01_Data_Exploration.ipynb`

---

## Stage 2 — Data Cleaning & Preparation
Tool: Python + Pandas + Excel / Google Sheets

Prepare the data for reliable analysis.

Tasks:
- Handle appropriate missing values
- Check duplicates
- Correct data types
- Standardize inconsistent values
- Validate dates
- Create useful derived columns
- Document important cleaning decisions

Output:
`02_Data_Cleaning.ipynb`

---

## Stage 3 — Data Modeling / Database Preparation
Tool: SQL

Understand how the different datasets connect and prepare them for
relational analysis.

Potential tables:
- Orders
- Customers
- Order Items
- Products
- Sellers
- Payments
- Reviews
- Geolocation

Tasks:
- Understand primary and foreign keys
- Create database tables
- Load cleaned data
- Establish relationships
- Validate row counts and relationships

Output:
SQL schema and database

---

## Stage 4 — Business Analysis with SQL
Tool: SQL

Answer meaningful business questions using the connected datasets.

Examples:
- How is revenue changing over time?
- Which product categories perform best?
- Which sellers generate the most revenue?
- What is the average order value?
- Which regions contribute the most orders?
- How does delivery performance vary?
- Is customer satisfaction related to delivery performance?

Skills:
- SELECT
- WHERE
- GROUP BY
- JOIN
- CASE
- Subqueries
- CTEs
- Window Functions

Output:
`03_SQL_Business_Analysis.sql`

---

## Stage 5 — Exploratory & Statistical Analysis
Tool: Python + Pandas + Statistics

Perform deeper analysis of the business questions identified earlier.

Tasks:
- Exploratory data analysis
- Distribution analysis
- Trend analysis
- Customer analysis
- Product analysis
- Seller analysis
- Delivery analysis
- Correlation analysis
- Appropriate statistical tests

Output:
`04_Statistical_Analysis.ipynb`

---

## Stage 6 — Business Intelligence Dashboard
Tool: Power BI

Turn the most important findings into an interactive business dashboard.

Possible sections:
- Executive Overview
- Sales Performance
- Customer Analysis
- Product / Seller Performance
- Delivery & Customer Satisfaction

The dashboard should focus on answering business questions rather than
simply displaying large numbers of charts.

Output:
Power BI dashboard

---

## Stage 7 — Machine Learning
Tool: Python + Scikit-learn

Add a machine learning component only where it provides genuine business
value.

Possible use cases:
- Customer segmentation
- Customer behavior prediction
- Delivery-related prediction
- Sales / demand prediction

The model should include:
- Problem definition
- Feature preparation
- Train/test split
- Model training
- Evaluation
- Interpretation of results

Output:
`05_Machine_Learning.ipynb`

---

## Stage 8 — Business Insights & Recommendations

Combine the findings from the analysis into actionable conclusions.

For each important finding:

1. What did we discover?
2. Why does it matter?
3. What could the business do about it?

The final recommendations should be supported by data.

Output:
Business findings / final report

---

## Stage 9 — GitHub Portfolio

Organize the complete project into a professional GitHub repository.

The repository should contain:
- Notebooks
- SQL files
- Documentation
- Dashboard
- README
- Relevant outputs

The README should explain:
- Business problem
- Dataset
- Methodology
- Tools used
- Key findings
- Business recommendations
- Project results

---

# Main Principle

The project follows this philosophy:

Understand → Clean → Connect → Analyze → Visualize → Predict → Recommend

Every technical step should support a meaningful business question.

Do not add complexity simply to make the project look advanced.