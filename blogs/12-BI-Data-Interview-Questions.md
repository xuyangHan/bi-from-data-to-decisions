# BI, Data Analyst, and Data Engineer Interview Questions (and How to Answer Them)

Interviews for BI, data analyst, and data engineering roles often feel unpredictable.

One interviewer asks about funnels and retention, another dives into modeling, and a third asks you to debug conflicting reports. Different titles, similar topics, but each role emphasizes something slightly different.

A useful way to think about most interviews is this:

> SQL skills -> Data thinking -> System understanding

If you can answer across all three layers, you usually do well.

This post gives you a practical question bank, what interviewers are really testing, and clear answer structures you can adapt to your own projects.

---

## 1. How Roles Differ (and Overlap)

Before the questions, anchor on what each role optimizes for:

- **BI / Data Analyst**
  - Core focus: clarity and decisions.
  - Day-to-day: metric definitions, SQL analysis, dashboards, stakeholder communication.
- **Data Engineer**
  - Core focus: reliability and scale.
  - Day-to-day: ingestion, transformations, modeling layers, data quality, pipeline operations.

The overlap is real. Most modern analytics interviews include parts of both.

---

## 2. Layer 1: SQL and Data Manipulation (Core Foundation)

These questions are almost guaranteed. The hard part is not syntax, it is correctness.

See also:

- [SQL for Analysis](./02-SQL-for-analysis.md)
- [Building Reliable Metrics](./03-Building-Reliable-Metrics.md)
- [Real BI Analysis in SQL](./04-Real-BI-Analysis-in-SQL.md)

### Common foundational questions (with traps)

- What is the difference between `COUNT(*)` and `COUNT(DISTINCT ...)`?
  > *Sample answer: `COUNT(*)` counts rows, while `COUNT(DISTINCT user_id)` counts unique users. If one user has multiple events, `COUNT(*)` will be larger. I pick based on grain: rows for event volume, distinct IDs for entity-level metrics.*
- When would `LEFT JOIN` vs `INNER JOIN` change your result?
  > *Sample answer: `INNER JOIN` keeps only matched rows, so it can drop users/orders with missing related records. `LEFT JOIN` preserves the left table and is safer when I need complete coverage, then I handle nulls explicitly.*
- What is a window function, and when would you use one?
  > *Sample answer: A window function calculates across related rows without collapsing detail. I use it for ranking, running totals, and first/last event logic, like finding each user’s first purchase date.*
- How do you calculate a running total?
  > *Sample answer: I use `SUM(metric) OVER (PARTITION BY group ORDER BY date)` so each row includes cumulative value up to that point. Then I validate ordering and date completeness.*
- What is the difference between `WHERE` and `HAVING`?
  > *Sample answer: `WHERE` filters rows before aggregation, while `HAVING` filters groups after `GROUP BY`. If I want only categories with revenue above 10k, that belongs in `HAVING`.*

### Common BI-style SQL questions

- Write a query to calculate daily active users (DAU).
  > *Sample answer: I define active behavior first (for example, `login` or `session_start`), group by event date, and compute `COUNT(DISTINCT user_id)` per day. I also align timezone to business reporting time.*
- How would you compute retention (Day 1 / Day 7 / Day 30)?
  > *Sample answer: I build cohorts by signup date, then check whether each cohort user returns on day 1, 7, and 30. Retention is returning users divided by cohort size at user-level grain.*
- Write a query for top 3 products per category.
  > *Sample answer: I aggregate revenue by category and product, then use `ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC)` and filter rank <= 3.*
- How do you calculate conversion rate correctly?
  > *Sample answer: I define numerator and denominator at the same grain and timeframe, usually distinct users or sessions. Then I verify joins do not duplicate either side and sanity-check by segment.*

### What they are really testing

- Grain awareness
- Aggregation correctness
- Handling duplicates and join inflation
- Ability to explain query logic in business terms

### Question example — “How do you calculate conversion rate correctly?”

**What they are testing**

Whether you align numerator and denominator at the same grain.

**Answer outline**

- Define exactly what converts and over what time window.
- Confirm both numerator and denominator are at the same entity grain (usually user/session).
- Guard against duplicate rows after joins.
- Explain how you validate with a quick segment-level sanity check.

---

## 3. Layer 2: Analytics Thinking (Where Most People Fail)

This layer separates “can write SQL” from “can make decisions with data.”

See also:

- [Exploratory Data Analysis](./07-Exploratory-Data-Analysis.md)
- [Python Data Visualization](./08-Python-Data-Visualization.md)
- [Building Reliable Metrics](./03-Building-Reliable-Metrics.md)

### 3.1 Metric definition questions

- How do you define revenue?
  > *Sample answer: I start by business intent, then define revenue components clearly: whether gross or net, tax treatment, discounts, refunds, and recognition timing. I document examples so finance and product interpret it the same way.*
- What is the difference between active user vs engaged user?
  > *Sample answer: Active user typically means any qualifying activity in a period, while engaged user means deeper behavior like multiple sessions or key feature usage thresholds. I define both explicitly because they answer different questions.*
- How would you define retention?
  > *Sample answer: Retention is the share of a cohort that returns after initial use within a specified window. I always specify cohort start, return event, and day window (D1, D7, D30) to avoid ambiguity.*

**What they want**

- Clarity
- Consistency
- Awareness of ambiguity and edge cases

**Strong answer pattern**

- Start with business intent.
- Give an unambiguous definition (population, event, timeframe).
- Call out exclusions and edge cases (refunds, bots, test users, timezone).
- Show how you would document it so teams stay aligned.

### 3.2 Metric debugging questions

- A dashboard shows revenue increased 20%. How do you validate it?
  > *Sample answer: I first verify data freshness and pipeline status, then compare underlying tables, filters, and joins versus prior logic. I decompose the increase by segment and run spot checks to confirm it is business-driven, not a calculation artifact.*
- Two reports show different numbers. What do you check first?
  > *Sample answer: I compare metric definitions and filters before anything else, then check date windows, timezone, and grain. Most mismatches come from definition drift or join logic differences.*

**Strong answer usually includes**

- Check data source and refresh timing.
- Check joins and potential duplication.
- Check grain alignment.
- Check filters and date logic.

### 3.3 Product/business diagnosis questions

- Why did conversion drop last week?
  > *Sample answer: I validate the metric, then break conversion into funnel steps and segment by channel, device, and user type. I test hypotheses in order, like traffic-quality shifts, UX changes, or checkout failures.*
- How would you analyze a drop in DAU?
  > *Sample answer: I confirm event tracking integrity first, then analyze by source, platform, geography, and cohort. I separate acquisition issues from engagement issues to isolate the primary driver.*
- What metrics would you track for a new feature?
  > *Sample answer: I track adoption, activation, engagement depth, retention impact, and downstream business impact. I include guardrail metrics like error rate and latency so we do not optimize one area while hurting another.*

**What they want**

- Structured thinking
- Hypothesis -> validation loop
- Segmentation by channel/region/cohort/device

### Question example — “Why did conversion drop last week?”

**Answer outline**

1. Validate the metric definition first.
2. Confirm the drop is real (not a pipeline or freshness issue).
3. Decompose into traffic x conversion x order value or equivalent levers.
4. Segment the change (channel, device, geo, new vs returning users).
5. Form hypotheses, test quickly, and summarize likely drivers and next actions.

---

## 4. Layer 3: System Understanding (Modeling + Warehousing)

Mid-level and senior interviews often probe this layer even for analyst roles.

See also:

- [Data Modeling for BI](./09-Data-Modeling-for-BI.md)
- [Data Warehousing](./10-Data-Warehousing.md)

### 4.1 Data modeling questions

- What is a fact table vs dimension table?
  > *Sample answer: A fact table stores measurable events, like orders or sessions, and dimensions provide descriptive context like customer, product, and date. Facts are usually large and numeric, dimensions are lookup-oriented and descriptive.*
- What does grain mean?
  > *Sample answer: Grain is what one row represents. If grain is one row per order line, every metric must respect that level, or aggregation errors happen quickly.*
- What is a star schema?
  > *Sample answer: A star schema places a central fact table with surrounding dimension tables. It simplifies BI queries and keeps definitions consistent for reporting tools.*
- Why not just query OLTP tables directly?
  > *Sample answer: OLTP systems are optimized for transactions, not heavy analytics. Querying them directly can hurt app performance and leads to inconsistent logic, so we use warehouse models for stable analysis.*

### 4.2 Real-world modeling questions

- How would you design a table for tracking user activity?
  > *Sample answer: I use an event fact table keyed by user and timestamp with event type and core attributes, plus dimensions for user/date/platform. I define clear event taxonomy and partition by date for performance.*
- How would you model orders and customers?
  > *Sample answer: I create `fact_orders` at order grain with customer and product keys, then `dim_customer`, `dim_product`, and `dim_date`. I define revenue fields carefully, including discount and refund handling.*

**What they want**

- Clear grain definition
- Correct relationships and keys
- Simplicity and query-friendliness

### 4.3 Data engineering / warehousing questions

- What is a data warehouse?
  > *Sample answer: A data warehouse is a centralized analytical store that integrates data from multiple systems and supports consistent, high-performance reporting and analysis.*
- What is ETL vs ELT?
  > *Sample answer: ETL transforms before loading, while ELT loads raw data first and transforms inside the warehouse. Modern cloud stacks often prefer ELT for flexibility and scalability.*
- What are stages of a data pipeline?
  > *Sample answer: Typical stages are source extraction, ingestion, raw storage, transformation/modeling, quality checks, and consumption through dashboards or data products.*
- What is the difference between OLTP and OLAP?
  > *Sample answer: OLTP handles many small writes for operations; OLAP handles large read-heavy analytical queries. BI should run on OLAP to avoid performance and consistency problems.*

### 4.4 Practical system questions

- How would you design a pipeline from app DB -> dashboard?
  > *Sample answer: I replicate app data to a warehouse, keep raw snapshots, build tested transformation layers, and expose curated marts to BI tools. I include freshness monitoring and ownership for each model.*
- How do you handle late-arriving data?
  > *Sample answer: I use incremental models with watermark logic and reprocessing windows so late records are captured safely. I also make downstream reports aware of data finalization lag.*
- How do you ensure data quality?
  > *Sample answer: I set automated checks for freshness, null/key constraints, volume anomalies, and business rule validity. Failures trigger alerts and controlled rollback or rerun paths.*

**What they want**

- End-to-end thinking
- Trade-off awareness (latency, cost, complexity)
- Reliability mindset

---

## 5. Common Pitfall Questions (Sneaky but Important)

These questions test whether you have been burned by real data problems.

See also:

- [Building Reliable Metrics](./03-Building-Reliable-Metrics.md)
- [Data Modeling for BI](./09-Data-Modeling-for-BI.md)

- Why do joins sometimes inflate metrics?
  > *Sample answer: Many-to-many or one-to-many joins can duplicate rows before aggregation. I prevent this by pre-aggregating, validating join cardinality, and checking row counts before and after joins.*
- What happens if you aggregate at the wrong grain?
  > *Sample answer: Metrics become biased, often overstated or understated, because entities are counted multiple times or merged incorrectly. I always state the row grain first before calculating KPIs.*
- When can `DISTINCT` hide problems instead of fixing them?
  > *Sample answer: `DISTINCT` can mask bad joins by removing duplicate symptoms without fixing root cause. I use it intentionally only when distinct entities are the true metric definition.*
- Why can ratios be misleading?
  > *Sample answer: Ratios mislead when numerator and denominator are from different grains, periods, or populations. I align both sides and report sample sizes and segment context.*

### What they are really testing

- Whether you can spot silent correctness failures
- Whether you debug root causes instead of patching symptoms

### Good response pattern

- Explain the failure mode.
- Give a concrete example.
- Show prevention (correct grain, pre-aggregation, key constraints, QA checks).

---

## 6. Case and Open-Ended Questions (Most Important)

These simulate real work and often matter more than trivia.

### Case 1 — “Revenue dropped 15% this month. What would you do?”

**Strong structure**

1. Validate the metric definition and data freshness.
2. Check overall trend and compare against seasonality.
3. Segment by region, product, channel, and cohort.
4. Form hypotheses and test in order of likely impact.
5. Summarize findings, confidence level, and recommended actions.

> *Sample answer: First I confirm revenue definition and that data is complete for the month. Then I compare month-over-month and year-over-year trends, segment by channel, region, and product, and isolate where the decline is concentrated. I test likely hypotheses such as traffic quality shifts, conversion issues, and pricing/refund changes. Finally I present top drivers with confidence levels and immediate actions, like fixing checkout issues or reallocating spend.*

### Case 2 — “Design a dashboard for a product team.”

**What they expect**

- Core KPIs (DAU, retention, conversion, activation).
- Trends over time and baseline comparisons.
- Segmentation (new vs returning, platform, channel).
- Clear metric definitions and alerting logic.

> *Sample answer: I would build a dashboard with activation, DAU/WAU, retention, and conversion to key actions, each with clear definitions. I would include trend charts, week-over-week comparisons, and segment filters for platform and user type. I would add alerts for sudden drops and a short metric glossary so product and analytics stay aligned.*

### Case 3 — “You have messy CSV + API + database data. What do you do?”

**What they expect**

- Ingestion strategy for each source.
- Cleaning and type standardization.
- Modeling into analytics-friendly tables.
- Automation, testing, and monitoring so the process is repeatable.

> *Sample answer: I ingest each source into a raw layer with lineage metadata, standardize schemas and types, and deduplicate with business keys and timestamps. Then I model analytics tables around defined grain and tested metrics. I schedule the flow, add data quality tests and alerts, and document assumptions so it is reliable beyond one-off analysis.*

---

## 7. One Practical Interview Framework

When stuck, use this sequence:

1. Clarify the business question.
2. Define metric and grain.
3. Validate data quality and freshness.
4. Analyze with segmentation and comparisons.
5. Recommend action with trade-offs.

This works for BI, analyst, and many data engineer interview prompts.

---

## 8. How to Prepare with This Blog Series

Use this post as a study map, not a script to memorize.

- Pick one layer per day: SQL, analytics thinking, or systems.
- Answer 3 questions out loud and write one concise response per question.
- For one question, sketch the SQL or table design you would use.
- For one case question, practice a 2-minute executive summary.

Then go deeper in these posts:

- BI fundamentals: [What is Business Intelligence?](./01-what-is-bi.md)
- SQL and metrics: [SQL for Analysis](./02-SQL-for-analysis.md), [Building Reliable Metrics](./03-Building-Reliable-Metrics.md), [Real BI Analysis in SQL](./04-Real-BI-Analysis-in-SQL.md)
- Python and analysis workflow: [Python for BI](./05-Python-for-BI.md), [Data Wrangling in Python](./06-Data-Wrangling-in-Python.md), [Exploratory Data Analysis](./07-Exploratory-Data-Analysis.md), [Python Data Visualization](./08-Python-Data-Visualization.md)
- Data architecture: [Data Modeling for BI](./09-Data-Modeling-for-BI.md), [Data Warehousing](./10-Data-Warehousing.md)
- Experimentation: [A/B Testing for BI](./11-AB-Testing.md)

If you can explain these questions clearly and tie them to real examples, you will be in a strong position for BI, data analyst, and data engineering interviews.

