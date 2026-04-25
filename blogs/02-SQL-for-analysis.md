# SQL for BI Analysis: Thinking in Metrics, Grain, and Questions

Teach *how to think before writing SQL*.

In BI work, SQL is not the goal. It is a tool for answering business questions with confidence.  
This post focuses on the thinking process that prevents wrong conclusions before you type your first `SELECT`.

---

## 0. SQL basics

``` sql
-- Retrieve all users older than 25
SELECT * FROM users WHERE age > 25;

-- Insert a new user named 'Alice', age 30
INSERT INTO users (name, age) VALUES ('Alice', 30);

-- Update Alice's age to 31
UPDATE users SET age = 31 WHERE name = 'Alice';

-- Delete users younger than 20
DELETE FROM users WHERE age < 20;

-- Join users and orders tables to get all orders with user information
SELECT users.name, orders.order_id, orders.total
FROM users
JOIN orders ON users.user_id = orders.user_id;

-- Group by age and get the number of users in each age group
SELECT age, COUNT(*) AS user_count
FROM users
GROUP BY age;

-- Order users by name in ascending order
SELECT * FROM users
ORDER BY name ASC;

-- Limit the results to 10
SELECT * FROM users
LIMIT 10;

-- Find the average age of users
SELECT AVG(age) AS average_age
FROM users;
```

## 1. Why BI SQL is different from “normal SQL”

In application engineering, SQL is often about retrieving data efficiently for a feature.
In BI, SQL is about producing numbers people will use to make decisions.

That changes your priority:

- not query writing -> decision support
- correctness > complexity
- clarity > cleverness

A short, readable query that defines a metric correctly is more valuable than a complex query that is hard to validate.

---

## 2. Start with the business question

Strong BI analysis starts with a concrete question:

- "Why did revenue drop last week?"
- "Which users are active this month?"

👉 translate into:

Before writing SQL, convert the question into three parts:

- metric: what exactly is measured (`revenue`, `active_users`, `conversion_rate`)
- entity: what each row should represent (`order`, `user`, `session`)
- time window: the analysis period (`last 7 days`, `current month`, `Q1`)

If any of these are unclear, stop and clarify. Most BI rework happens because definitions were assumed, not agreed.

---

## 3. The most important concept: Grain

**Grain** means the row-level meaning of a dataset.

Examples:

- one row per order
- one row per user per day
- one row per product per month

Why it matters: aggregation only works when grain is explicit.
If your metric is at user level but your table is at event level, you can accidentally count the same user many times.

Wrong grain = wrong answers, even if the SQL runs perfectly.

A good habit is to state grain in one sentence before analysis:
`This table has one row per ___`.

---

## 4. Dimensions vs Measures

BI queries become easier when you separate:

- dimensions: attributes used to segment data (`country`, `plan_type`, `channel`, `date`)
- measures: numeric values to aggregate (`revenue`, `orders`, `sessions`)

Ask yourself:

- "Am I grouping by dimensions?"
- "Am I aggregating measures?"

Mixing these roles creates messy logic and ambiguous outputs. Clean separation leads to cleaner dashboards.

---

## 5. Mental model of BI SQL

Raw data → structured view → metric → insight

Think in this pipeline:

1. Raw data: events, transactions, logs
2. Structured view: cleaned, deduplicated, clearly defined grain
3. Metric: explicit calculation aligned with business definition
4. Insight: interpretation tied to the original question

If the structured view is weak, every downstream metric is fragile.
Most "SQL mistakes" are actually modeling and definition mistakes.

---

## 💡 Key takeaway

> Most BI SQL mistakes happen before writing SQL — not inside it.

---
When you slow down and define **question, metric, grain, and time window** first, your SQL becomes simpler, faster to review, and much more trustworthy.
