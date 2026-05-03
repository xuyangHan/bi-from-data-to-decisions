# Data Warehousing for BI

If BI is about making better decisions, data warehousing is the system that makes those decisions trustworthy and repeatable.

In real companies, data comes from many places: product databases, marketing platforms, payment systems, spreadsheets, and logs. Without a warehouse, every team builds reports differently, numbers do not match, and leadership loses confidence in analytics.

This post explains the core ideas behind data warehousing in a practical way: what it is, how it works end to end, and what trade-offs teams make in production.

## 1) Why Data Warehousing Matters in BI

Most companies hit the same problems before they invest in a warehouse:

- Different dashboards show different values for the same KPI.
- Analysts spend more time cleaning data than analyzing it.
- Reporting is slow because data is scattered across tools.
- Business teams cannot answer cross-functional questions quickly.

A data warehouse solves this by centralizing analytical data and standardizing logic. Instead of asking, "Whose number is correct?", teams align on one definition and one source.

In practice, the warehouse becomes the analytics backbone:

- Finance trusts revenue numbers.
- Product teams track behavior consistently.
- Marketing evaluates campaign impact with comparable metrics.
- Leadership gets faster, cleaner answers.

## 2) OLTP vs OLAP 

Already covered this in the previous post, so here is the practical recap:

- `OLTP` systems run day-to-day operations (orders, payments, user updates).
- `OLAP` systems run analytical workloads (large scans, aggregations, trends).

Why this matters: your production app database is built for many small transactions, not heavy dashboard queries. If BI runs directly on OLTP, performance suffers on both sides: reports are slow and application workload can be affected.

Rule of thumb:

- Use OLTP to run the business.
- Use OLAP to understand the business.

## 3) Data Warehouse Architecture (End-to-End Flow)

A typical warehouse architecture has five stages:

1. **Source systems**: Where data is created: application databases, SaaS tools, CSV files, and event logs.
2. **Ingestion**: Data is moved into the warehouse, either in batches (hourly/daily) or in real time.
3. **Raw layer**: Data is stored in its original form with minimal changes, so you can trace issues back to the source if needed.
4. **Transformation/modeling**: Data is cleaned, joined, and structured into analytics-friendly tables (such as fact and dimension tables).
5. **Consumption**: The final data is used in dashboards, SQL queries, and business reports.

```mermaid
flowchart LR
  sourceSystems[SourceSystems] --> ingestLayer[IngestionLayer]
  ingestLayer --> rawZone[RawZone]
  rawZone --> transformLayer[TransformLayer]
  transformLayer --> modeledLayer[ModeledLayer]
  modeledLayer --> biConsumption[BIDashboardsAndReports]
```

The key design idea is separation of concerns:

- Keep raw data for auditability and backfills.
- Keep modeled data for business-friendly analysis.
- Keep BI consumption focused on trusted tables, not raw source schemas.

## 4) ETL vs ELT 

Both ETL and ELT are valid. The right choice depends on context.

`ETL` means transform first, then load:

- Useful when strict preprocessing is required before storage.
- Often used in legacy/on-prem architectures.
- Can become rigid as requirements change.

`ELT` means load first, then transform inside the warehouse:

- Fits modern cloud warehouses with scalable compute.
- Preserves raw data and makes reprocessing easier.
- Speeds up iteration when business logic evolves.

Why ELT became common: cloud platforms made storage cheaper and warehouse compute more flexible, so teams can load broadly and model incrementally.

Practical decision criteria:

- Choose ETL when governance demands strict pre-load controls.
- Choose ELT when agility, reproducibility, and scale are priorities.

## 5) Data Modeling for Analytics 

As you covered in the modeling post, this is where many BI outcomes are won or lost.

At a high level:

- **Fact tables** capture measurable events (orders, sessions, transactions).
- **Dimension tables** provide context (customer, product, date, region).
- **Star schemas** keep analysis intuitive and performant.

Two concepts to keep front and center:

- **Grain**: what one row in a fact table represents.
- **Metric definitions**: consistent business logic for KPIs.

If grain is unclear or KPI logic is inconsistent, dashboards may look polished but still mislead decisions.

## 6) Modern Data Stack Overview

The stack usually includes these capability layers, and each layer answers a different question:

- How do we collect data?
- Where do we store and compute on it?
- How do we clean and model it?
- How do we run pipelines reliably?
- How do business users consume trusted metrics?
- How do we monitor quality, freshness, and cost?

### 6.1 Ingestion

- Replicate data from OLTP databases (PostgreSQL, MySQL, SQL Server) into the warehouse.
- Capture SaaS data (Salesforce, HubSpot, Stripe, Google Ads) on a schedule.
- Ingest event streams (app clicks, product telemetry, logs).
- Land raw files (CSV, JSON, parquet) from partners or internal systems.

**Common tools**

- Fivetran, Airbyte, Stitch (managed ELT connectors)
- Kafka, Kinesis, Pub/Sub (streaming/event ingestion)
- S3, GCS, Azure Blob + batch loaders (file-based ingestion)

**Example**

- A retail team syncs Shopify orders every 15 minutes, streams web events in near real time, and lands nightly supplier CSV files into a raw zone.

### 6.2 Warehouse/Storage + Compute

- Store raw and modeled datasets in a central analytical platform.
- Separate storage and compute so workloads can scale independently.
- Optimize partitioning/clustering and query performance for BI workloads.

**Common tools**

- Snowflake, BigQuery, Redshift (cloud data warehouses)
- Databricks lakehouse on Delta Lake / Apache Iceberg (warehouse + data lake patterns)

**Example**

- Finance dashboards run on a dedicated compute warehouse, while data science jobs run on separate clusters so workloads do not compete.

### 6.3 Transformation/Modeling

- Standardize raw source data into clean staging models.
- Build business-ready marts (sales, finance, product, support).
- Define reusable metrics and dimensions with version-controlled SQL.
- Add tests for nulls, uniqueness, and referential integrity.

**Common tools**

- dbt (SQL transformations, testing, documentation)
- SQL/Python transformations in warehouse-native workflows
- Dataform or custom frameworks (depending on team preferences)

**Example**

- `stg_orders` cleans source order records, `fct_orders` becomes the canonical fact table, and `dim_customer` powers customer segmentation across dashboards.

### 6.4 Orchestration/Scheduling

- Trigger and coordinate dependencies between ingestion and transformation jobs.
- Manage retries, backfills, SLAs, and failure alerts.
- Schedule pipelines (hourly, daily, event-driven) with observability into runs.

**Common tools**

- Airflow, Dagster, Prefect (pipeline orchestration)
- Managed schedulers in cloud platforms (for simpler workflows)

**Example**

- A daily DAG runs at 6:00 AM: ingest ERP data -> run dbt models -> refresh BI extracts -> send Slack status to stakeholders.

### 6.5 BI/Semantic Layer

- Build dashboards, ad hoc exploration, and KPI scorecards.
- Define semantic logic so "revenue" or "active customer" has one trusted definition.
- Apply row-level security for department or region-specific access.

**Common tools**

- Power BI, Tableau, Looker, Metabase, Superset (BI and dashboarding)
- LookML, dbt semantic models, or metric layers for consistent definitions

**Example**

- Marketing, finance, and leadership all use the same net revenue metric, but each team has a tailored dashboard view.

### 6.6 Monitoring/Observability

- Detect freshness delays, row count anomalies, schema drift, and failed tests.
- Track pipeline reliability and incident response metrics.
- Monitor warehouse spend and query efficiency.

**Common tools**

- Monte Carlo, Soda, Great Expectations (data quality/observability)
- Native warehouse monitoring + cloud cost dashboards
- Alerting integrations (Slack, PagerDuty, Teams)

**Example**

- If daily order volume drops by 40% vs historical baseline, the pipeline sends an alert and blocks dashboard refresh until data is validated.

It is easy to over-focus on vendor names. In practice, architecture quality matters more than specific tools:

- Is data reliable?
- Is logic reusable?
- Can teams ship changes safely?
- Can costs stay predictable as data grows?

A reference stack is useful for learning, but the best stack is the one that fits your team skills, data volume, compliance constraints, and budget.

## 7) Getting Data Warehousing Right

A warehouse is only valuable if people trust it, and trust comes from handling trade-offs deliberately instead of chasing tools.

Common mistakes usually start here:

- Treating tool choice as strategy.
- Ignoring data modeling and shared metric definitions.
- Waiting too long to add quality checks.
- Mixing raw and business-ready tables without clear boundaries.
- Overengineering early before core KPIs are stable.

These mistakes become more likely when teams face normal trade-offs:

**Cost vs performance**

- Faster queries often mean more compute spend.
- Aggressive optimization can reduce flexibility.

**Speed vs quality**

- Shipping quickly builds momentum.
- Skipping tests creates long-term trust debt.

**Central BI vs self-service**

- Centralized teams improve consistency.
- Self-service increases business agility.
- Most organizations need a hybrid model.

**Build vs buy**

- Building custom pipelines can fit unique needs.
- Buying managed tools accelerates delivery.
- Total cost includes maintenance, not just licensing.

Practical controls help teams move fast without losing trust:

- Automated tests for freshness, uniqueness, and referential integrity.
- Documentation for models, columns, and business logic.
- Data lineage so teams can trace dashboard metrics to source.
- Clear ownership for each pipeline and domain table.

Governance essentials keep the platform safe and sustainable:

- Role-based access control.
- Sensitive data handling (PII, financial data).
- Compliance-aware retention and auditing policies.

Modern teams also add:

- **Data contracts** between producers and consumers.
- **Observability** for failures, anomalies, and SLA monitoring.

In short, good warehouse programs balance speed, cost, and consistency while making quality visible and enforceable.


## Final Thoughts

Good BI is not about having the fanciest stack. It is about creating a trusted analytical system that helps people make better decisions repeatedly.

If you remember one idea from this post, make it this: build your warehouse so that business logic is clear, data quality is visible, and metrics are consistent across teams. Everything else is optimization.
