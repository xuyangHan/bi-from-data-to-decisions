# SQL for BI Analysis: Thinking in Metrics, Grain, and Question

Teach *how to think before writing SQL*.

In BI work, SQL isn’t the goal, but it’s just the tool.
The real goal is answering business questions **correctly and confidently**.

This post focuses on the thinking process that helps you avoid wrong conclusions *before* you write your first `SELECT`.

![SQL for BI](../assets/azure_2_sqltopbi.png)

---

## 0. Do you need SQL basics?

If you’re already comfortable with `SELECT`, `JOIN`, and `GROUP BY`, feel free to skip this section.

Otherwise, here’s a quick refresher:

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

---

## 1. Why BI SQL is different from “normal SQL”

In application development, SQL is often about powering features efficiently.

In BI, SQL is about producing numbers people will **trust and act on**.

That changes your priorities:

* not query writing → **decision support**
* not “does it run” → **is it correct**
* not cleverness → **clarity**

A simple query that defines a metric clearly is far more valuable than a complex one that nobody can confidently verify.

---

## 2. Start with the business question

Good BI work starts with a clear question:

* “Why did revenue drop last week?”
* “Which users are active this month?”

Before writing SQL, break the question into three parts:

* **metric** → what are we measuring? (`revenue`, `active_users`)
* **entity (grain)** → what does each row represent? (`order`, `user`, `session`)
* **time window** → when are we measuring? (`last 7 days`, `this month`)

If any of these are unclear, pause.

Most BI rework doesn’t come from bad SQL, but it comes from **unclear definitions**.

---

## 3. The most important concept: Grain

**Grain** is the meaning of one row in your dataset.

Examples:

* one row per order
* one row per user per day
* one row per product per month

Why this matters:

Aggregation only works when grain is clear.

If your table is at the *event level* but your metric is at the *user level*, you can accidentally count the same user multiple times.

👉 Simple mistake:

```sql
SELECT COUNT(user_id) FROM events;
```

This counts *events*, not users.

Correct version:

```sql
SELECT COUNT(DISTINCT user_id) FROM events;
```

Same table. Same column. Completely different answer.

A good habit:

> Always state the grain before you write SQL:
> “This table has one row per ___”

---

## 4. Dimensions vs Measures

BI queries get much easier when you separate two roles:

* **dimensions** → how you slice data (`country`, `plan_type`, `date`)
* **measures** → what you calculate (`revenue`, `orders`, `users`)

Ask yourself:

* Am I grouping by **dimensions**?
* Am I aggregating **measures**?

Mixing these leads to confusing queries and hard-to-interpret results.

Clean separation → cleaner logic → clearer outputs.

---

## 5. Mental model of BI SQL

Think of BI work as a pipeline:

**Raw data → structured view → metric → insight**

* **Raw data** → events, logs, transactions
* **Structured view** → cleaned, deduplicated, well-defined grain
* **Metric** → clearly defined calculation
* **Insight** → tied back to a real question

If the structured layer is weak, everything downstream becomes fragile.

Most “SQL mistakes” aren’t really SQL problems, but they’re **modeling and definition problems**.

---

## 💡 Key takeaway

> Most BI SQL mistakes happen before writing SQL, not inside it.

When you slow down and define:

* the **question**
* the **metric**
* the **grain**
* the **time window**

your SQL becomes simpler, easier to review, and much more reliable.
