# Olist E-Commerce Analytics Project

Olist is a Brazilian marketplace that connects small sellers to customers. This project investigates where the business is growing, where customers may be getting lost, and how delivery performance is associated with reviews and repeat purchases, then turns the findings into recommendations a non-technical manager can act on.

---

## 1\. Business Context (the "story" of the project)

**Who am I working for?** A fictional *Head of Operations & Growth* at Olist.

**What do they care about?** Three things:

1. **Growth:** Is the business growing, and where does the money come from?  
2. **Customers:** Do customers come back? Are they happy?  
3. **Operations:** Do sellers and deliveries meet customer expectations, and what does it cost when they do not?

**The main business problem I am solving:**

Olist's customers rarely buy a second time, and a share of reviews are negative. How is delivery performance associated with customer reviews and repeat purchases, which sellers and regions stand out, and what should the business do about it?

Everything in the project connects back to this problem. Every chart, query, and model must help answer a question that a manager would actually ask.

---

## 2\. Dataset

**Source:** Olist Brazilian E-Commerce Public Dataset (Kaggle). Real, anonymized orders from 2016 to 2018\.

| Table | What one row represents | Key columns |
| :---- | :---- | :---- |
| `orders` | One order | `order_id`, `customer_id`, `order_status`, purchase / approval / carrier / delivery / estimated-delivery timestamps |
| `customers` | One customer *per order* | `customer_id`, `customer_unique_id`, zip prefix, city, state |
| `order_items` | One product line inside an order | `order_id`, `order_item_id`, `product_id`, `seller_id`, `price`, `freight_value` |
| `order_payments` | One payment record for an order | `order_id`, `payment_type`, `payment_installments`, `payment_value` |
| `order_reviews` | One review | `review_id`, `order_id`, `review_score` (1 to 5), comment, dates |
| `products` | One product | `product_id`, category (Portuguese), weight, dimensions |
| `sellers` | One seller | `seller_id`, zip prefix, city, state |
| `geolocation` | Many lat/long points per zip prefix | zip prefix, latitude, longitude |
| `product_category_name_translation` | Portuguese to English category names | two columns |

&nbsp;

### Things I must understand early (these are common traps)

- `customer_id` is unique **per order**. `customer_unique_id` identifies the **real person**. Using the wrong one breaks all repeat-customer analysis.  
- One order can have **many items** and **many payment rows**. Joining carelessly will **inflate revenue** (a classic "fan-out" mistake).  
- `geolocation` has many rows per zip prefix, so it must be aggregated before joining.  
- Some orders are cancelled, unavailable, or never delivered, so they must be treated differently from delivered orders.  
- The 2016 and late-2018 data are sparse, so trend analysis must be restricted to the reliable period.  
- A few orders may have reviews missing or duplicated, and some delivered orders may have missing delivery dates.

I will verify each of these myself in Stage 1 and write down what I find. I will not assume.

---

## 3\. Core Business Questions

These are the main business questions for the project. They are grouped by theme, and each one is tagged with the stage where it is answered. I will prioritize the core questions first and treat advanced statistics and machine learning as optional extensions if time is limited.

### A. Growth and Revenue

1. How have orders and revenue changed month by month? Is growth real, or is it seasonal (e.g. Black Friday)? *(SQL, Dashboard)*  
2. Which product categories bring the most revenue, and which have the highest average order value? *(SQL, Dashboard)*  
3. How concentrated is revenue? Do the top 20% of sellers or categories generate about 80% of sales (Pareto analysis)? *(SQL, Python)*  
4. Which states and cities drive demand, and is the business too dependent on São Paulo? *(SQL, Dashboard)*

### B. Customers and Retention

5. What percentage of customers ever buy a second time? *(SQL, Python)*  
6. Do customers acquired in different months behave differently over time (cohort retention)? *(SQL, Python)*  
7. Who are the most valuable customers (RFM: Recency, Frequency, Monetary), and what do they look like? *(Python, ML)*  
8. Do payment choices (credit card, boleto, installments) relate to order value? *(SQL, Python)*

### C. Delivery and Operations

9. How long does delivery actually take, and how accurate are the estimated delivery dates? *(SQL, Python)*  
10. What percentage of orders arrive late, and which states, sellers, and categories are the worst? *(SQL, Dashboard)*  
11. Where is the delay happening: seller handover to the carrier, or carrier to the customer? *(SQL, Python)*  
12. How much does shipping cost (freight) eat into order value, and how does it vary by region and product weight? *(SQL, Python)*

### D. Customer Satisfaction

13. Are late deliveries associated with worse review scores, and by how much? *(Python, Statistics)*  
14. Is the drop in satisfaction gradual (1 day late, 5 days late, 10+ days late) or a sudden cliff? *(Python, Statistics)*  
15. Which categories and sellers have consistently low ratings even when delivery is on time (i.e. a *product or seller quality* problem, not a delivery problem)? *(SQL, Python)*  
16. Can we predict which orders are likely to receive a bad review before it happens? *(ML)*

### E. Seller Performance

17. Which sellers combine high revenue with poor delivery or poor reviews (high risk for the platform)? *(SQL, Dashboard)*  
18. Are a small number of sellers responsible for a big share of all late deliveries? *(SQL, Python)*

> **Rule:** If a question cannot lead to an action the business could take, drop it.

---

## 4\. Tools and Why Each One Is Used

| Tool | Used for | Why it matters to an employer |
| :---- | :---- | :---- |
| Python \+ Pandas | Exploring, cleaning, statistics, ML | Core analyst skill |
| Excel / Google Sheets | Quick sanity checks, a pivot table to cross-check one SQL result | Shows I can validate my own work |
| **PostgreSQL** (recommended) | Database, joins, business queries | SQL is in almost every analyst job description |
| Power BI | Interactive dashboard | The most requested BI tool at entry level |
| Scikit-learn | Segmentation and prediction | Shows I know when ML adds value |
| Git \+ GitHub | Version control, portfolio | Recruiters will actually look at this |

&nbsp;

**Notes**

- PostgreSQL is recommended because Power BI connects to it directly. SQLite is easier to install but needs extra setup for Power BI.  
- Power BI Desktop runs on Windows. If I do not have Windows access, I will plan an alternative (Power BI Service in the browser, or Tableau Public / Looker Studio) before Stage 6, not during it.

---

## 5\. Project Workflow

**Philosophy:** Understand, Clean, Connect, Analyze, Visualize, Predict, Recommend.

For every stage, I will write a **"Plain-English Summary"** (2 to 4 sentences) at the top of the notebook or file. A non-technical reader should understand *what was done and why* without reading the code.

---

### Stage 0: Setup (half a day)

**Tasks**

- Create the GitHub repo and folder structure (see Section 8).  
- Download the dataset into `data/raw/` and never edit these files.  
- Create a `docs/data_dictionary.md` that I fill in as I learn the data.  
- Create a `docs/decision_log.md`. Every cleaning or analysis choice goes here with a one-line reason.  
- Add `data/` to `.gitignore` if the files are large, and add a download note in the README.

**Output:** Empty but organized repo with a first commit.

---

### Stage 1: Data Exploration

**Tool:** Python \+ Pandas **Plain-English goal:** Get to know the data before touching it.

**Tasks**

- Load all 9 files. For each, record shape, dtypes, missing values, duplicate rows, and unique counts.  
- Check key uniqueness: Is `order_id` unique in `orders`? How many `customer_unique_id` values vs `customer_id` values?  
- Check date ranges and month-by-month order counts to find the reliable period.  
- Look at `order_status` counts.  
- Look for impossible values: delivery before purchase, price of 0, extreme weights, review dates before the order date.  
- Count orders with multiple items, multiple sellers, and multiple payment rows.

**Questions this stage must answer in writing**

- What is the grain (one row \= what) of each table?  
- Which joins are safe and which cause fan-out?  
- Which time period should be used for analysis?

**Output:** `notebooks/01_Data_Exploration.ipynb` \+ completed `data_dictionary.md`

---

### Stage 2: Data Cleaning and Preparation

**Tool:** Python \+ Pandas (+ Excel/Sheets for a manual spot check) **Plain-English goal:** Make the data trustworthy and add the columns needed for analysis.

**Tasks**

- Convert timestamp columns to proper datetime.  
- Translate category names to English; label missing categories as `unknown` instead of dropping them.  
- Decide what to do with non-delivered orders (keep for some analyses, exclude for delivery analysis) and log it.  
- Handle duplicate reviews (keep the most recent per order) and document it.  
- Aggregate `geolocation` to one row per zip prefix (median lat/long).  
- Validate: no negative delivery times, no missing delivered-order dates left unexplained.

**Derived columns to create** (these power the whole project)

| Column | Meaning |
| :---- | :---- |
| `order_month` | Month of purchase |
| `delivery_days` | Delivered date minus purchase date |
| `estimated_days` | Estimated date minus purchase date |
| `delay_days` | Delivered minus estimated (positive means late) |
| `is_late` | 1 if delivered after the estimated date |
| `delay_bucket` | On time / 1-3 days late / 4-7 / 8+ |
| `seller_handling_days` | Carrier pickup date minus approval date |
| `carrier_transit_days` | Delivered date minus carrier pickup date |
| `order_value` | Sum of item prices (and separately, including freight) |
| `freight_ratio` | Freight divided by item price |
| `is_low_review` | 1 if review score is 1 or 2 |
| `customer_state`, `seller_state`, `same_state` | Whether buyer and seller are in the same state |
| `seller_customer_distance_km` | Haversine distance from zip-level coordinates (optional, adds depth) |

&nbsp;

**Validation habit:** after cleaning, the total revenue from my cleaned table must match the raw `order_items` total. If it does not, I have a join bug.

**Output:** `notebooks/02_Data_Cleaning.ipynb`, cleaned CSVs in `data/processed/`, updated `decision_log.md`

---

### Stage 3: Data Modeling and Database

**Tool:** PostgreSQL **Plain-English goal:** Store the data in a database where tables are connected by keys, like a real company would.

**Tasks**

- Create the database and tables with proper data types.  
- Define primary keys, composite keys (e.g. `order_items` uses `order_id` \+ `order_item_id`), and foreign keys.  
- Draw the ER diagram from the actual keys and relationships confirmed during exploration; save it to `docs/er_diagram.png`.  
- Load the cleaned CSVs.  
- Validate: row counts in SQL match pandas, and there are no orphan foreign keys (e.g. items whose order does not exist).  
- Create a few **views** that pre-join common data, for example `vw_order_summary` (one row per order with revenue, delay, and review) so later queries stay readable and safe from fan-out.

**Output:** `sql/01_schema.sql`, `sql/02_load_data.sql` (or a documented load script), `sql/03_validation_checks.sql`, ER diagram in `docs/`

---

### Stage 4: Business Analysis with SQL

**Tool:** SQL **Plain-English goal:** Answer the business questions directly from the database.

**Query list (progressive difficulty, so I learn as I go)**

*Level 1: Basics (SELECT, WHERE, GROUP BY, ORDER BY)*

- Total orders, revenue, unique customers, sellers.  
- Orders by status.  
- Revenue by payment type.

*Level 2: JOINs and CASE*

- Top 10 categories by revenue (join items, products, translation).  
- Late vs on-time orders by state with `CASE`.  
- Average review score by delay bucket.

*Level 3: CTEs and Subqueries*

- Customers with 2+ orders (using `customer_unique_id`) and the repeat purchase rate.  
- Sellers whose late rate is above the platform average.  
- Orders where the seller delay is larger than the carrier delay.

*Level 4: Window Functions (the "I put in real effort" level)*

- Month-over-month revenue growth with `LAG()`.  
- Rank sellers within each state by revenue with `RANK()` / `DENSE_RANK()`.  
- Running total of revenue and cumulative share of sellers (Pareto) with `SUM() OVER`.  
- Cohort retention table: first purchase month vs months since first purchase.  
- RFM scoring with `NTILE()`.  
- Each customer's time between first and second order.

**Rules for every query**

- A comment above it states the **business question** and the **finding** in one sentence.  
- Check that totals reconcile with Stage 3 validation (no fan-out).

**Output:** `sql/04_business_analysis.sql` (organized by question number from Section 3), plus `docs/sql_findings.md` summarizing results in plain English.

---

### Stage 5: Exploratory and Statistical Analysis

**Tool:** Python (Pandas, Matplotlib/Seaborn, SciPy/Statsmodels) **Plain-English goal:** Go deeper than SQL and check whether the patterns are *real* or just noise.

**Analyses**

- **Distributions:** delivery time, order value (likely skewed, so I will use median alongside mean and explain why), freight ratio.  
- **Trend and seasonality:** monthly orders and revenue; flag the November 2017 spike.  
- **Customer analysis:** repeat rate, time to second order, spend per customer.  
- **Delivery analysis:** estimated vs actual delivery, late rate by state, seller handling vs carrier transit time.  
- **Satisfaction analysis:** review score distribution (expect it to be U-shaped, with many 5s and many 1s) and what drives the 1s.  
- **Seller and category analysis:** late rate and low-review rate by seller and category, with a minimum order count (e.g. 30+) to avoid judging sellers with 2 orders.

**Statistical tests (each must be explained in plain English)**

| Question | Test | Plain-English meaning |
| :---- | :---- | :---- |
| Do late orders get lower review scores? | Mann-Whitney U (scores are ordinal, not normal) | "Is the difference real or luck?" |
| Is lateness related to getting a low review (1-2 stars)? | Chi-square test \+ odds ratio | "How many times more likely is a bad review when late?" |
| Do review scores differ across delay buckets? | Kruskal-Wallis | "Does it get worse the later it is?" |
| Is delivery time related to distance or freight? | Spearman correlation | "Do these move together?" |
| How certain is the late rate? | Bootstrap or proportion confidence interval | "Our estimate is X%, give or take Y" |

&nbsp;

I will report **effect size** (how big the difference is), not just p-values, because a tiny difference can be "significant" with 100k rows but still not matter to the business.

**Output:** `notebooks/04_Statistical_Analysis.ipynb`, key charts saved to `outputs/figures/`

---

### Stage 6: Business Intelligence Dashboard

**Tool:** Power BI **Plain-English goal:** Give a manager one place to see the answers and explore them.

**Data connection:** connect to PostgreSQL views (or the processed CSVs if needed). Build a small star-schema style model: one fact table (order items) with date, customer, product, and seller dimension tables.

**Pages (4 pages maximum, quality over quantity)**

1. **Executive Overview** KPI cards: Revenue, Orders, Average Order Value, Repeat Customer %, Late Delivery %, Average Review Score. Monthly revenue trend. Revenue by state map.  
2. **Sales and Products** Top categories, category share, AOV by category, Pareto of revenue.  
3. **Delivery and Satisfaction** Late % by state, estimated vs actual delivery, review score by delay bucket (the "headline" chart), seller handling vs carrier time.  
4. **Sellers and Risk** Table of sellers by revenue, late %, and review score, with a highlighted "high revenue, high risk" group.

**Requirements**

- Slicers: Year / Month, State, Category.  
- DAX measures written for the KPIs (e.g. Late %, Repeat %, AOV) and documented.  
- Every page has a title that states the **takeaway**, e.g. "Late deliveries are associated with X-point lower average review scores" instead of "Delivery Analysis". Avoid wording that implies causation unless the analysis design supports it.  
- Consistent colors, no clutter, readable by someone with no analytics background.  
- Export screenshots and a PDF of the dashboard to the repo (recruiters may not open `.pbix` files).

**Output:** `dashboard/olist_dashboard.pbix`, `dashboard/screenshots/`, `dashboard/dashboard_guide.md` (how to read each page)

---

### Stage 7: Machine Learning

**Tool:** Python \+ Scikit-learn **Rule:** Only two models, both tied to a business decision. No complexity for show.

#### Model A: Customer Segmentation (unsupervised)

- **Business use:** Marketing can treat different customer groups differently.  
- **Approach:** Build RFM features per `customer_unique_id`, scale them, run K-Means, and choose k using the elbow method and silhouette score.  
- **Honest caveat:** Most Olist customers buy only once, so RFM is limited. I will explicitly report that and interpret the segments carefully (this is an insight in itself).  
- **Output:** Named segments (e.g. "One-time high spenders", "Recent single buyers", "Repeat customers") with a profile table.

#### Model B: Predict Bad Reviews (supervised classification)

- **Business use:** Flag risky orders early (for example, a long estimated delivery to a far state with a slow seller) so support or logistics can act before the customer complains.  
- **Target:** `is_low_review` (1-2 stars).  
- **Features:** order value, freight ratio, product weight/size, estimated delivery days, seller-customer distance, same-state flag, seller's historical late rate, payment type/installments, product category.  
- **Avoid data leakage:** do **not** use actual delivery date or actual delay as features if the goal is *early* prediction. I will build two versions and compare:  
  - *Version 1 (early warning):* only information known at purchase time.  
  - *Version 2 (post-delivery):* includes actual delay. This is mainly used to confirm that delay is the driver.  
- **Process:** train/test split (stratified), baseline (logistic regression), then a tree-based model (random forest or gradient boosting), handling class imbalance (class weights).  
- **Evaluation:** precision, recall, F1, ROC-AUC, confusion matrix. Accuracy alone is misleading with imbalance, and I will explain why.  
- **Interpretation:** feature importance, plus a plain-English statement of what the model suggests (and what it does not prove).

**Output:** `notebooks/05_Machine_Learning.ipynb`

---

### Stage 8: Business Insights and Recommendations

**Plain-English goal:** Turn analysis into decisions.

For every key finding I will use this template:

|  |  |
| :---- | :---- |
| **What did we find?** | A specific number, e.g. "Orders delivered 8+ days late average X stars vs Y stars when on time." |
| **Why does it matter?** | Link to revenue, retention, or reputation. |
| **What should the business do?** | A concrete action. |
| **How would we measure success?** | A KPI and a target direction. |
| **Confidence / limits** | Sample size, assumptions, what the data cannot tell us. |

&nbsp;

**Example recommendation types to aim for (to be backed by my actual results, not assumed)**

- Adjust estimated delivery dates for slow regions so promises are realistic.  
- Put underperforming high-revenue sellers on an improvement plan, and reward reliable sellers.  
- Target retention campaigns at specific customer segments or categories.  
- Reduce dependence on a few states, categories, or sellers.  
- Use the bad-review early-warning model to trigger proactive customer communication.

**Output:** `docs/Business_Report.md` (and optionally a 5-8 slide PDF summary). Written for a manager, not a data scientist.

---

### Stage 9: GitHub Portfolio

**README must include**

1. Project title and a one-sentence business problem.  
2. "Key Findings" at the **top** (3 to 5 bullets with real numbers and a dashboard screenshot). Recruiters skim.  
3. Dataset description and the data-quality traps I found.  
4. Methodology (the workflow in Section 5, briefly).  
5. Tools used.  
6. Business recommendations.  
7. How to reproduce (folder guide, environment setup, run order).  
8. Limitations and next steps.

**Quality checklist**

- Notebooks run top to bottom without errors and have Markdown explanations between code cells.  
- No hard-coded local paths. Use relative paths.  
- `requirements.txt` included.  
- Meaningful commit messages (one commit per stage or meaningful step, not one giant commit).  
- Screenshots render in the README.

---

## 6\. Suggested Timeline (adjust to your pace)

| Week | Focus | Learning alongside |
| :---- | :---- | :---- |
| 1 | Stage 0, 1, and 2 | Pandas refresher (merge, groupby, datetime) |
| 2 | Stage 3 | PostgreSQL install, keys, joins basics |
| 3 | Stage 4 (levels 1 to 3\) | SQL practice daily (SELECT, JOIN, CASE, CTE) |
| 4 | Stage 4 (level 4\) \+ findings notes | Window functions |
| 5 | Stage 5 | Hypothesis testing basics |
| 6 | Stage 6 | Power BI basics, DAX measures, data modeling |
| 7 | Stage 7 | Scikit-learn workflow, evaluation metrics |
| 8 | Stage 8 and 9 | Writing, README polish |

&nbsp;

If time is short, protect the **core path**: Stages 1, 2, 3, 4, 6, 8, 9\. Stage 5 and 7 can be trimmed (e.g. only the late-vs-review tests and one ML model).

---

## 7\. Learning Strategy (since SQL and Power BI are new)

- **Learn by need, not in advance.** Before each stage, spend 1 to 2 days on the specific concept, then apply it immediately.  
- **Write the business question first, then the query.** This builds the habit employers want.  
- **Explain every result out loud or in writing** in one plain sentence. If I cannot, I do not understand it yet.  
- **After each stage, write a short learning checkpoint:** what I did, why I did it, what I learned, and one thing I can explain confidently in an interview.  
- **Separate findings from interpretation:** describe what the data shows, what it suggests, and what it cannot prove. Observational data can show associations, but does not by itself prove that delivery delays cause poor reviews or fewer repeat purchases.  
- **Cross-check results** between SQL, pandas, and a quick Excel pivot for at least a few key numbers (total revenue, repeat rate, late %).  
- **Keep a "mistakes I made" section** in `decision_log.md` (fan-out joins, wrong customer ID, etc.). It is excellent interview material.

---

## 8\. Repository Structure

```
olist-ecommerce-analytics/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/                 # original files (not edited)
│   └── processed/           # cleaned files
├── notebooks/
│   ├── 01_Data_Exploration.ipynb
│   ├── 02_Data_Cleaning.ipynb
│   ├── 04_Statistical_Analysis.ipynb
│   └── 05_Machine_Learning.ipynb
├── sql/
│   ├── 01_schema.sql
│   ├── 02_load_data.sql
│   ├── 03_validation_checks.sql
│   └── 04_business_analysis.sql
├── dashboard/
│   ├── olist_dashboard.pbix
│   ├── screenshots/
│   └── dashboard_guide.md
├── docs/
│   ├── data_dictionary.md
│   ├── decision_log.md
│   ├── er_diagram.png
│   ├── sql_findings.md
│   └── Business_Report.md
└── outputs/
    └── figures/
```

&nbsp;

---

## 9\. Definition of "Done"

The project is complete when:

- [ ] All core business questions have written answers supported by actual results; complete the remaining questions if time allows.  
- [ ] SQL totals reconcile with pandas totals.  
- [ ] The dashboard has 4 pages, each with a takeaway title and working slicers.  
- [ ] If Stage 5 is completed, at least 3 statistical tests are explained in plain English with effect sizes.  
- [ ] If Stage 7 is completed, both ML models have a clear business use, a leakage check, and honest evaluation.  
- [ ] The business report gives at least 5 data-backed recommendations.  
- [ ] The README lets a stranger understand the project in 2 minutes.  
- [ ] I can explain every step in an interview without reading my notes.

---

## 10\. CV and Interview Notes

**CV bullet templates** (fill in with *real* numbers from my results, never invent them):

- Analyzed \~100K real e-commerce orders (Olist) using Python, PostgreSQL, and Power BI to identify drivers of customer dissatisfaction and low repeat purchases.  
- Wrote advanced SQL (CTEs, window functions, cohort retention, RFM scoring) to answer 18 business questions on revenue, delivery performance, and seller risk.  
- Showed with statistical testing that late deliveries lower review scores by **\[X\]** points and raise the odds of a 1-2 star review by **\[Y\]x**.  
- Built a 4-page Power BI dashboard for executives covering sales, customers, delivery, and seller risk.  
- Developed a classification model to flag orders at risk of bad reviews (ROC-AUC **\[Z\]**) and segmented customers with K-Means; delivered data-backed recommendations.

**Interview questions this project prepares me for**

- What is the difference between `customer_id` and `customer_unique_id`, and what would have gone wrong if you mixed them up?  
- How did you avoid double-counting revenue when joining tables?  
- Why did you use the median or a non-parametric test here?  
- What is data leakage, and where could it have happened in your model?  
- What would you recommend to the business, and how would you measure whether it worked?

---

## Main Principle

> Understand → Clean → Connect → Analyze → Visualize → Predict → Recommend

Every technical step must support a business question. Depth comes from **careful reasoning and clear explanation**, not from extra tools or complicated models. If a step does not help answer a question in Section 3, it does not belong in the project.