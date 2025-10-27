# 🟠 dbt Models — Key Concepts

## Writing Models & Best Practices
- A **dbt model** is just a `SELECT` statement, usually wrapped in a CTE.  
- Create a **folder for each layer** (staging → intermediate → marts).  
- Extend functionality with **Jinja** (`{% if %}`, loops, macros).  
- Models can also be written in **Python** (in supported warehouses).  
- To view compiled SQL (requires dbt VS Code extension):  
  **Ctrl + Shift + P → Compile model**  
- Models can **reference each other** using `ref()` and `source()`.  
- dbt manages **environments, schemas, and lineage** automatically.

### Best Practices
- Default to **views** in staging and **tables** in marts.  
- Maintain clear folder hierarchy: **staging → intermediate → marts**.
- **Staging:**  
  - 1:1 with source tables  
  - Light transformations only (rename, cast, clean)  
  - No joins or aggregations  
  - Naming convention: `stg_[source]__[table]s.sql`  
- **Intermediate:**  
  - Centralize complex logic  
  - Reusable building blocks (joins, aggregations)  
  - Naming convention: `int_[entity]s_[verb]s.sql`  
- **Marts:**  
  - Business-ready entities (facts/dims)  
  - Organized by business area (`marts/finance`, `marts/marketing`)  
- Use **incremental** materialization for large fact tables where rebuilds are costly.  
- Keep **ephemeral models** small — too many can bloat compiled SQL.  
- Always define sources in `sources.yml` (avoid hardcoding schemas).  
- Use `cluster_by` and `partition_by` for large datasets to optimize performance.  

---

## ref()
- A core feature that lets you reference one dbt model from another.  
- Use `{{ ref('model_name') }}` instead of hardcoding table names.  
- dbt automatically compiles refs to the correct **schema** and **table name** for the environment.  
- Improves **readability** and makes SQL code cleaner.  

Example:
```sql
select * from {{ ref('orders') }}
```

---

## source()
- Use `{{ source('source_name', 'table_name') }}` to define raw sources.  
- Makes models **environment-agnostic** and easier to maintain.  

Example:
```sql
select * from {{ source('raw_sales', 'transactions') }}
```

---

## Materializations

```sql
{{ config(
  materialized = 'table' | 'view' | 'incremental' | 'ephemeral'
) }}
```

### Types
- **Table** → creates or overwrites a physical table.  
- **View** → creates or overwrites a view.  
- **Incremental** → appends or updates only changed records (performance boost for large data).  
- **Ephemeral** → exists only at compile time (inline CTE).  
  - Simplifies logic without persisting results.  
  - Good for transformations reused by multiple models but not needed in the warehouse.

### Materialized View (⚠️ Not supported in Snowflake)
- A pre-computed table storing query results — a hybrid between a table and a view.  
- The database handles refresh automatically.  
- For **incremental** models, you must explicitly define how to find and merge new data.  
- More info: [dbt Materializations Docs](https://docs.getdbt.com/docs/build/materializations)

---

## Model Configurations — Common Parameters

Example:
```sql
{{ config(
  tags = ['daily'],
  enabled = false,
  schema = 'Tom',
  pre_hook = "USE DATABASE DEV_DATABASE; CREATE SCHEMA IF NOT EXISTS Tom",
  cluster_by = ['sales_date']
) }}
```

### Common Config Parameters
- `enabled:` enable/disable a model  
- `tags:` assign model tags for grouping and orchestration  
- `pre_hook:` SQL to run before model execution  
- `post_hook:` SQL to run after model execution  
- `database:` override default database  
- `schema:` override default schema  
  - dbt creates schema as `<main_schema>_<custom_name>`  
- `cluster_by:` clustering keys (Snowflake)  
- `partition_by:` partitioning configs (BigQuery)  
- `liquid_clustered_by:` clustering configs (Databricks)  
