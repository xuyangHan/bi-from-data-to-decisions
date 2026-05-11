# Python Basics for BI: Bridging Excel, SQL, and Modern Data Analytics

If you’ve ever wondered why everyone in analytics is suddenly talking about Python, or felt unsure about how it fits between your trusty Excel sheets and complex SQL queries, this guide is for you. By the end, you’ll see exactly *why* Python unlocks a new level of power for BI analysts, no matter your starting point.

![Python Basics for BI](../assets/python-BI.png)

---

## 1. Why BI Needs Python Beyond Excel and SQL

Let’s be honest: Excel and SQL alone aren’t going anywhere. They’re the foundation of day-to-day BI work. But every analyst eventually runs into the same problems:

- **Excel gets overwhelmed fast.** Try working with millions of rows or repeating the same steps for a weekly report. Spreadsheets break, formulas get murky, and nothing is truly repeatable.
- **SQL is amazing for querying, limited for everything else.** Need to pull data from an API? Merge dozens of files? Build complex transformation pipelines or automate EDA (exploratory data analysis)? SQL alone can’t do it all.
- **Enter Python:** Python acts like the “glue” for all your analytical work. It connects databases, files, APIs, and analytical code, all in one script. It doesn’t replace Excel or SQL; it turbocharges them.

---

## 2. Where Does Python Fit in the BI Workflow?

Python isn’t just another tool, but it’s a Swiss Army knife for analysts. Here’s how it fits into real work:

- **Data Extraction:** Open CSVs, fetch web data, connect to APIs. Python handles all sources, no matter the size or format.
- **Data Transformation:** With libraries like Pandas, you can clean, join, filter, and shape your data at any scale, and unlike in Excel, every step is reproducible.
- **Analysis (EDA):** Slice and dice datasets programmatically, calculate KPIs and metrics, and ask “what if?” questions quickly, all inside Python.
- **Visualization:** Build everything from quick charts to interactive dashboards using libraries like matplotlib, seaborn, or plotly.
- **Automation:** Want your weekly dashboard to land in a stakeholder’s inbox every Monday morning, automatically? Python scripts make it happen.

---

## 3. Python & Pandas for BI: The Practical Cheat Sheet

This is your consolidated, quick-start reference for the most essential Python *and* Pandas operations you’ll use daily as a BI analyst, featuring real code and comments. Scan here for most tasks, from variables to repeatable analytics.

### Python & Pandas BI Cheat Sheet

#### 1. CORE PYTHON BASICS

```python
# Variables
revenue = 15000
month = "January"

# Lists
sales = [120, 300, 450]
sales.append(600)

# Access list item
print(sales[0])

# Loop
for amt in sales:
    print(amt)

# Dictionary
row = {"Date": "2023-01-01", "Revenue": 200}
print(row["Revenue"])

# Function
def total_revenue(vals):
    return sum(vals)

print(total_revenue([10, 20, 30]))

# Conditional
amt = 2500

if amt > 1000:
    print("High")
else:
    print("Normal")
```

---

#### 2. PANDAS BASICS

```python
import pandas as pd

# Read CSV
df = pd.read_csv("sales.csv")

# Top rows
print(df.head())

# Column selection
df["Revenue"]

# Multiple columns
df[["Date", "Revenue"]]

# Filter rows
high_sales = df[df["Revenue"] > 10000]

# Save CSV
df.to_csv("output.csv", index=False)
```

---

#### 3. LOC vs ILOC (VERY COMMON INTERVIEW QUESTION)

##### loc → label/name based

```python
df.loc[0]                  # row with index label 0
df.loc[0, "Revenue"]       # specific cell
df.loc[:, ["Date", "Revenue"]]
```

Think:

> “Use actual row/column names”

---

##### iloc → integer position based

```python
df.iloc[0]                 # first row
df.iloc[0, 1]              # first row, second column
df.iloc[:, 0:2]
```

Think:

> “Use row/column positions”

---

##### Example

```python
# first 5 rows, first 2 columns
df.iloc[0:5, 0:2]

# rows where Revenue > 1000
df.loc[df["Revenue"] > 1000]
```

---

#### 4. FILTERING & BOOLEAN LOGIC

```python
# AND
df[(df["Revenue"] > 1000) & (df["Region"] == "West")]

# OR
df[(df["Region"] == "West") | (df["Region"] == "East")]

# isin
df[df["Region"].isin(["West", "East"])]

# NOT
df[~df["Region"].isin(["West"])]
```

---

#### 5. NULL / MISSING VALUES

##### Detect nulls

```python
df.isnull()
df["Revenue"].isnull()
```

##### Count nulls

```python
df.isnull().sum()
```

##### Drop nulls

```python
df.dropna()

# Drop rows if Revenue is null
df.dropna(subset=["Revenue"])
```

##### Fill nulls

```python
df["Revenue"] = df["Revenue"].fillna(0)
```

VERY common interview topic.

---

#### 6. SORTING

```python
# Ascending
df.sort_values("Revenue")

# Descending
df.sort_values("Revenue", ascending=False)

# Multiple columns
df.sort_values(["Region", "Revenue"])
```

---

#### 7. GROUPBY (MOST IMPORTANT PANDAS SKILL)

##### SQL equivalent

```sql
SELECT Region, SUM(Revenue)
FROM sales
GROUP BY Region
```

##### Pandas

```python
df.groupby("Region")["Revenue"].sum()
```

---

##### Multiple aggregations

```python
df.groupby("Region").agg({
    "Revenue": ["sum", "mean", "max"],
    "CustomerID": "count"
})
```

---

#### 8. APPLY & LAMBDA FUNCTIONS

##### Create calculated column

```python
df["Tax"] = df["Revenue"] * 0.13
```

##### Apply custom logic

```python
df["Category"] = df["Revenue"].apply(
    lambda x: "High" if x > 1000 else "Low"
)
```

Interviewers LOVE this one.

---

#### 9. STRING OPERATIONS

```python
# lowercase
df["Name"].str.lower()

# contains
df[df["Email"].str.contains("@gmail.com")]

# replace
df["Phone"].str.replace("-", "")
```

---

#### 10. DATETIME OPERATIONS

SUPER important in BI.

```python
df["Date"] = pd.to_datetime(df["Date"])
```

##### Extract parts

```python
df["Year"] = df["Date"].dt.year
df["Month"] = df["Date"].dt.month
df["Day"] = df["Date"].dt.day
```

##### Filter by date

```python
df[df["Date"] >= "2024-01-01"]
```

---

#### 11. MERGING / JOINS

##### SQL INNER JOIN equivalent

```python
merged = pd.merge(
    customers,
    orders,
    on="CustomerID",
    how="inner"
)
```

##### Join types

```python
how="left"
how="right"
how="outer"
how="inner"
```

Very common interview question:

> “Explain left join vs inner join.”

---

#### 12. DUPLICATES

```python
# Find duplicates
df.duplicated()

# Remove duplicates
df.drop_duplicates()

# Remove based on subset
df.drop_duplicates(subset=["CustomerID"])
```

---

#### 13. VALUE COUNTS

```python
df["Region"].value_counts()
```

Equivalent to frequency analysis.

---

#### 14. RENAME COLUMNS

```python
df.rename(columns={
    "Rev": "Revenue"
})
```

---

#### 15. COLUMN CREATION / TRANSFORMATION

```python
df["Profit"] = df["Revenue"] - df["Cost"]
```

---

#### 16. ITERROWS (Usually Avoid)

```python
for index, row in df.iterrows():
    print(row["Revenue"])
```

But interview tip:

> pandas vectorized operations are preferred over loops for performance.

Good thing to mention.

---

#### 17. PERFORMANCE / BEST PRACTICES

##### Prefer vectorized operations

GOOD:

```python
df["Tax"] = df["Revenue"] * 0.13
```

BAD:

```python
for i in range(len(df)):
    df.loc[i, "Tax"] = df.loc[i, "Revenue"] * 0.13
```

---

#### 18. FILE HANDLING / AUTOMATION

```python
import os

for filename in os.listdir("data_folder"):
    if filename.endswith(".csv"):

        path = os.path.join("data_folder", filename)

        df = pd.read_csv(path)

        print(df.head())
```

---

#### 19. EXCEL FILES

```python
# Read Excel
df = pd.read_excel("sales.xlsx")

# Write Excel
df.to_excel("output.xlsx", index=False)
```

---

#### 20. COMMON INTERVIEW QUESTIONS

##### Q: loc vs iloc?

- `loc` → label based
- `iloc` → integer position based

---

##### Q: groupby does what?

Splits data into groups and applies aggregation.

---

##### Q: Why use pandas?

Efficient table/data analysis similar to SQL + Excel combined.

---

##### Q: Why vectorization matters?

Faster and more memory efficient than Python loops.

---

##### Q: Difference between merge and concat?

- `merge` → SQL-style join
- `concat` → stack/combine datasets

---

#### 21. REALLY USEFUL EXTRA ONES

##### unique values

```python
df["Region"].unique()
```

##### number of unique values

```python
df["CustomerID"].nunique()
```

##### column names

```python
df.columns
```

##### data types

```python
df.dtypes
```

##### dataframe info

```python
df.info()
```

##### summary stats

```python
df.describe()
```

**Use this cheat sheet as your go-to Python & Pandas reference for BI analysis. Most loading, filtering, summarizing, and scripting tasks start here: combining the best of programming and “Excel++” workflows.**

Pandas, at the heart of much of this, is what lets Python quietly start replacing the repetitive pain points you face in BI: constantly exporting the same SQL queries, copy-pasting new CSVs, or rebuilding Excel pivots by hand.

Here’s why it matters beyond just “another tool”:

- **Rows and Columns:** Gives you the familiar feel of SQL tables and Excel sheets, but with far more power. Every transformation is captured in code and can be rerun, making your work truly repeatable.
- **Easy Transformations and Aggregations:** Clean, filter, pivot, merge, and summarize your data just like you would in Excel or SQL, but with one reproducible script instead of dozens of manual steps or exports.
- **Scalability:** Handles datasets that choke Excel or require multiple SQL extracts, to process tens of millions of rows smoothly, all in memory and fully scriptable.
- **Bridges Excel and SQL:** Rather than switching back and forth (and losing track of steps), Pandas lets you pull data directly from SQL, manipulate it like a pro, and automate what used to be manual Excel “workflows.”

*With Pandas, you don’t just analyze data. You automate and document your analytics process, freeing yourself from repetitive SQL exports and endless Excel formulas. Each step is repeatable and reliable, so your BI work scales with your needs.*

---

## 💡 Takeaway

Python isn’t here to wipe out Excel or SQL, but it’s here to fill in their gaps, automate your workflow, and let you analyze data *your way.* Once you’re comfortable with the essentials above, you’ll be equipped to tackle any BI task, from wrangling messy data to building powerful, automated insights.
