# Marketing Analytics Dashboard
### End-to-End Analysis | SQL · R · Power BI

---

## Overview

This project presents an end-to-end marketing analytics solution built for an online retail business experiencing declining customer engagement and conversion rates despite actively running new marketing campaigns. The objective was to conduct a detailed analysis of the business's marketing and customer data, identify the root causes of underperformance, and deliver actionable recommendations through an interactive Power BI dashboard.

The full pipeline covers data extraction and transformation in **Microsoft SQL Server**, sentiment analysis in **R**, and final visualisation and reporting in **Power BI**.

---

## Business Problem

An online retail client approached with the following pain points:

- Declining customer engagement across marketing channels
- Falling conversion rates despite investment in new campaigns
- No structured mechanism for capturing or acting on customer feedback

The business needed clarity on *why* performance was dropping and *where* to focus improvement efforts.

---

## Business Goals

| # | Goal |
|---|------|
| 1 | Identify the factors impacting conversion rates and provide recommendations to improve them |
| 2 | Highlight key stages in the conversion funnel where customers drop off and suggest optimisations |
| 3 | Determine the type of content that drives the highest engagement |
| 4 | Analyse interaction levels across different marketing content types to inform future strategy |
| 5 | Understand common themes in customer reviews and surface actionable insights |
| 6 | Identify recurring positive and negative feedback to guide product and service improvements |

---

## Key Performance Indicators (KPIs)

- **Conversion Rate** — percentage of visitors completing a purchase
- **Customer Engagement** — interactions (views, clicks, likes) across content types
- **Average Order Value (AOV)** — mean spend per completed transaction
- **Customer Feedback Score** — derived from ratings and sentiment analysis

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Microsoft SQL Server | Data extraction, cleaning, and transformation |
| R (sentimentr, RODBC) | Sentiment analysis on customer reviews |
| Power BI | Interactive dashboard and visualisation |

---

## ETL Process

Data was extracted from the client's operational database into **Microsoft SQL Server** for transformation, then loaded into **Power BI** for analysis and visualisation.

### 1. Engagement Data — Cleaning & Normalisation

The `engagement_data` table contained several quality issues that required resolution before analysis:

- **Inconsistent content type labels** — `Socialmedia` was standardised to `Social Media` using `UPPER()` and `REPLACE()`
- **Combined views and clicks column** — a single `ViewsClicksCombined` field was split into separate `Views` and `Clicks` columns using `CHARINDEX()` and `LEFT()`/`RIGHT()` string functions
- **Date formatting** — `EngagementDate` was converted and reformatted to `dd.MM.yyyy` for consistency
- **Newsletter exclusion** — newsletter content type was filtered out as it fell outside the scope of this analysis

### 2. Customer Reviews — Whitespace Cleaning

The `ReviewText` column in the `customer_reviews` table contained double-space whitespace errors. A `REPLACE()` function was applied to normalise all review text before downstream sentiment processing.

### 3. Customer & Geography Join

A `LEFT JOIN` was performed between the `customers` table and the `geography` table on `GeographyID` to enrich each customer record with their associated `Country` and `City`. This enabled geographic segmentation of conversion data to identify where the majority of conversions were originating.

### 4. Product Price Categorisation

Products in the `products` table were categorised into three price tiers using a `CASE` statement:

| Category | Price Range |
|----------|------------|
| Low | Under £50 |
| Medium | £50 – £200 |
| High | Over £200 |

This segmentation was used to assess whether price point influenced engagement and conversion behaviour.

### 5. Customer Journey — Deduplication & Null Handling

The `customer_journey` table required the most significant cleaning:

- **Duplicate detection** — a Common Table Expression (CTE) with `ROW_NUMBER() OVER (PARTITION BY ...)` was used to identify duplicate journey records based on `CustomerID`, `ProductID`, `VisitDate`, `Stage`, and `Action`
- **Deduplication** — only the first occurrence of each duplicate group was retained by filtering on `row_num = 1`
- **Null duration handling** — missing `Duration` values were replaced with the average duration for that visit date using `COALESCE()` combined with `AVG() OVER (PARTITION BY VisitDate)`
- **Stage standardisation** — all `Stage` values were uppercased for consistency using `UPPER()`

---

## Sentiment Analysis (R)

Customer review data was exported from SQL Server into **R** via an ODBC connection for sentiment scoring and categorisation using the `sentimentr` package.

### Approach

Each review in the `ReviewText` column was passed through a custom sentiment scoring function that returned a numeric sentiment score. This score was then cross-referenced against the customer's star `Rating` to produce a nuanced sentiment category, capturing cases where written sentiment and rating score did not align.

### Sentiment Categories

| Category | Logic |
|----------|-------|
| **Positive** | Score > 0.05 and Rating ≥ 4 |
| **Mixed Positive** | Score > 0.05 but Rating = 3, or Score neutral but Rating ≥ 4 |
| **Neutral** | Score between -0.05 and 0.05 and Rating = 3 |
| **Mixed Negative** | Score < -0.05 but Rating = 3, or Score positive but Rating < 3 |
| **Negative** | Score < -0.05 and Rating ≤ 2 |

### Sentiment Buckets

Scores were also grouped into four numeric buckets for distribution analysis:

- `0.5 to 1.0` — Strongly positive
- `0.0 to 0.49` — Mildly positive
- `-0.49 to 0.0` — Mildly negative
- `-1.0 to -0.5` — Strongly negative

The enriched review dataset (with `SentimentScore`, `SentimentCategory`, and `SentimentBucket` columns) was exported back into Power BI for visualisation alongside the rest of the transformed data.

---

## Power BI Dashboard

The cleaned and enriched datasets were loaded into Power BI, where an interactive four-page dashboard was built to address each of the six business goals. All pages include year and month slicers for flexible time-period analysis, and a product filter for drill-down by individual SKU.

---

### Page 1 — Overview

![Overview Dashboard](<img width="1170" height="662" alt="overview" src="https://github.com/user-attachments/assets/941bf338-4301-4ab9-ba4c-8b9d558d93cc" />
<img width="1170" height="662" alt="overview" src="https://github.com/user-attachments/assets/941bf338-4301-4ab9-ba4c-8b9d558d93cc" />
)

The Overview page provides a high-level summary across all three performance dimensions — conversion, social media engagement, and customer reviews — enabling quick identification of overall trend direction before drilling into detail pages.

**Key metrics (2024):**
- Overall conversion rate: **8.5%**
- Social media views: **2,982,369** | Clicks: **458,345 (15%)** | Likes: **73,618 (2%)**
- Average customer rating: **3.67 / 5**
- Conversion rate peaks in **September (16%)** and **December (10.3%)**, with troughs in **April–May (4.5%)**
- **Ski Boots, Kayak, and Surfboard** are the top-converting products across the period

---

### Page 2 — Conversion Details

![Conversion Details](<img width="1177" height="647" alt="conversion" src="https://github.com/user-attachments/assets/1b2e746a-f3b4-4d2a-b99d-17bc238f8972" />
)

This page breaks down the customer journey funnel and conversion performance by product and month.

**Key metrics (2024):**
- Customer journey funnel: **672 Views → 355 Clicks → 185 Drop-offs → 57 Purchases**
- Overall conversion rate: **8.5%**
- Top converting products: **Ski Boots (20.7%)**, **Kayak (17.9%)**, **Surfboard (13.9%)**
- Lowest converting products: **Climbing Rope (2.7%)**, **Boxing Gloves (2.7%)**, **Cycling Helmet (2.9%)**
- Strong seasonal conversion peaks in **January (19.6%)**, **September (10.5%)**, and **December (16%)**

---

### Page 3 — Social Media Details

![Social Media Details](<img width="1170" height="656" alt="social media" src="https://github.com/user-attachments/assets/1c985776-2c49-4869-8bbc-75615b9af812" />
<img width="1170" height="656" alt="social media" src="https://github.com/user-attachments/assets/1c985776-2c49-4869-8bbc-75615b9af812" />
)

This page analyses engagement distribution across content types — Blog, Social Media, and Video — and tracks performance by product across the year.

**Key metrics (2025, January):**
- Views: **1,096,704** | Clicks: **67,632** | Likes: **4,342**
- Blog content drives the highest view volumes in early months (Jan–Mar), with Social Media and Video becoming more competitive from April onwards
- Views decline sharply from a January–February peak through to September–October before a slight recovery, suggesting campaigns are front-loaded in the year
- The engagement heatmap shows **Ski Boots (29.69%)**, **Boxing Gloves (28.15%)**, and **Basketball (21.48%)** capturing the largest share of January traffic

---

### Page 4 — Customer Review Details

![Customer Review Details](<img width="1180" height="658" alt="customer reviews" src="https://github.com/user-attachments/assets/06510023-4540-43f3-a471-365b360f418f" />
<img width="1180" height="658" alt="customer reviews" src="https://github.com/user-attachments/assets/06510023-4540-43f3-a471-365b360f418f" />
)

This page presents the full sentiment analysis output, combining star ratings with NLP-derived sentiment scores produced in R.

**Key metrics:**
- Average rating: **3.69 / 5**
- Sentiment distribution: **Positive: 840** | **Mixed Negative: 169** | **Mixed Positive: 160** | **Negative: 194**
- Recurring negative review themes: *"product stopped working after a month"*, *"terrible customer service"*, *"colour was different from what was shown"*
- Negative and mixed negative reviews are consistent throughout the year — indicating systemic product and service issues rather than isolated incidents
- Customers with low ratings (1–2) submit disproportionately more reviews, skewing the public feedback profile

---

## Key Findings & Recommendations

### Finding 1 — The Click-to-Purchase drop-off is the primary conversion bottleneck

Of 672 views, only 57 result in a purchase — a **91.5% overall drop-off**. The steepest decline occurs between **Click (355) and Drop-off (185)**, meaning customers are actively engaging with content but abandoning before completing a transaction. This points to a post-click experience problem rather than a visibility or reach problem.

> **Recommendation:** Audit the post-click journey — landing page relevance, page load speed, checkout friction, and trust signals (reviews, returns policy). A/B test a simplified checkout flow and ensure campaign landing pages directly reflect the content that drove the click.

---

### Finding 2 — Conversion is highly seasonal and concentrated in a small number of products

Conversion peaks in **January (19.6%)** and **September–December**, aligning with seasonal demand for outdoor and sports equipment. **Ski Boots (20.7%)**, **Kayak (17.9%)**, and **Surfboard (13.9%)** significantly outperform the 8.5% portfolio average. At the other end, **Climbing Rope**, **Boxing Gloves**, and **Cycling Helmet** all sit below 3%.

> **Recommendation:** Concentrate campaign budget on high-converting products during their seasonal peak windows. For consistently low-converting products, investigate whether the issue lies in pricing, product page quality, or a mismatch between audience targeting and actual purchase intent.

---

### Finding 3 — Content generates strong reach but weak engagement depth

The business generates nearly **3 million views** but achieves only a **15% click-through rate** and a **2% like rate**. Blog content dominates early-year view volumes but engagement drops materially from mid-year, with no content type sustaining momentum through Q3.

> **Recommendation:** The gap between views and clicks suggests content is reaching audiences who are not yet in a purchase mindset. Introduce more bottom-of-funnel content — product comparisons, customer testimonials, and time-limited offers — alongside top-of-funnel awareness content. Revisit the content calendar to distribute campaign activity more evenly across the year rather than concentrating spend in Q1.

---

### Finding 4 — Negative customer feedback is systemic, not isolated

Across all time periods, the most common negative review themes are **product durability** (*"stopped working after a month"*), **customer service quality** (*"terrible customer service, would not buy again"*), and **product accuracy** (*"colour was different from what was shown"*). The consistency of these themes across months confirms these are structural issues, not one-off incidents.

> **Recommendation:** Escalate durability complaints to the product and supply chain teams for quality review on the most-complained-about SKUs. Improve product imagery accuracy and introduce detailed specification guides on product pages. Implement a structured customer service response protocol for negative reviews to demonstrate responsiveness and limit reputational damage.

---

### Finding 5 — Dissatisfied customers are over-represented in the review pool

The scatter plot on the Customer Review page shows that customers with average ratings of 1–2 submit a disproportionately high volume of reviews compared to satisfied customers. This means the public-facing review profile is skewed more negatively than the true average rating of **3.69** suggests.

> **Recommendation:** Introduce a post-purchase follow-up sequence — via email or in-app notification — to actively encourage satisfied customers to leave reviews. Balancing the review pool will provide a more representative picture of product quality and improve the business's public perception without requiring any change to the products themselves.

---

## Repository Structure

```
marketing-analytics/
│
├── sql/
│   ├── 01_clean_engagement_data.sql
│   ├── 02_clean_customer_reviews.sql
│   ├── 03_customer_geography_join.sql
│   ├── 04_product_price_categories.sql
│   └── 05_deduplicate_customer_journey.sql
│
├── r/
│   └── sentiment_analysis.R
│
├── dashboard/
│   └── marketing_analytics.pbix
│
├── images/
│   ├── overview.png
│   ├── conversion.png
│   ├── social_media.png
│   └── customer_reviews.png
│
└── README.md
```

---

## Author

**Adetola Adekoya**
[linkedin.com/adetola-adekoya](https://linkedin.com/adetola-adekoya) · [Portfolio](https://adetolaadekoya.github.io/AdekoyaAdetola.github.io/)

