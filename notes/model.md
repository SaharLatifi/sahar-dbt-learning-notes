# 🟠 dbt Models — Key Concepts

## Writing Models
- A dbt model is just a **`SELECT` statement**, usually wrapped in a **CTE**.  
- You can extend functionality with **Jinja** (`{% if %}`, loops, macros).  
- Models can also be written in **Python** (in supported warehouses).  
- To see the compiled SQL sent to the warehouse:  
  `Ctrl + Shift + P → Compile model`

---

## ref
- A core feature that lets you reference one dbt model from within another. A smart shortcut that dbt uses to build the data transformation pipeline. Instead of hardcoding a table name we can use `{{ ref('model_name') }}` to link models and track dependencies.  
- Helps to manage different environments (like development and production) without changing your code. dbt automatically compiles {{ ref('model_name') }} to the correct table name in the correct schema for the environment you're working in. 
-  Readability  -> It makes the SQL code cleaner and easier to read. 

## source
- Use `{{ source('source_name', 'table_name') }}` to define raw sources.  
- This makes models **environment-agnostic** and improves maintainability.  

---

## Materializations
Models can be materialized in four main ways:

- **Table** → creates/overwrites a physical table  
- **View** → creates/overwrites a view  
- **Incremental** → appends/updates only changed records (performance boost for large data)  
- **Ephemeral** → acts as an inline CTE (exists only at compile time)  
  - Use when you need to **simplify logic without persisting** intermediate results  
  - Good for transformations reused by multiple models but not needed in the warehouse  
- ** Materialized View ** (Not supported in Snowflake)
A pre-computed table that stores the results of a query. It's like a mix between a table and a view, designed to give you the speed of a table and the freshness of a view. The database itself handles the refresh. You don't have to write any specific logic for this. but for incremental model, You have to write the logic in your dbt SQL to tell it what new data to find and how to merge it
https://docs.getdbt.com/docs/build/materializations

---

## Model Configurations — Common Parameters & Best Practices

```yaml
models:
  my_project:
    +materialized: table
    +schema: analytics
    +tags: ['daily', 'core']


## Model Configurations — Common Parameters & Best Practices

### Common Config Parameters
- `enabled:` enable/disable a model  
- `tags:` assign model tags for grouping - jobs and orchestration
- `pre-hook:` SQL to run before model execution  
- `post-hook:` SQL to run after model execution  
- `database:` override default database  
- `schema:` override default schema  - dbt creates a new schema using the main schema + '_' + new schema name
- `cluster_by:` clustering keys (Snowflake)  
- `partition_by:` partitioning configs (BigQuery)  
- `liquid_clustered_by:` clustering configs (Databricks)  

### Best Practices
- Default to **views** in staging and **tables** in marts.  
- staging:  1:1 with source tables, light transformation/rename/cast/clean/no join or aggregations, naming: stg_[source]__[table]s.sql
- interediate: centralize complex logic,break down complexity, reusable building blocks, complex joins and aggregations, naming: int_[entity]s_[verb]s.sql
- marts: business-ready entities for consumption, final layer, dim/facts, organiza by business areas (marts/finance, marts/marketing)
- Use **incremental** only for large fact tables where rebuilds are costly.  
- Keep **ephemeral models** small; too many can bloat compiled SQL.  
- Always define sources in `sources.yml` instead of hardcoding schemas.  
- Organize models into **staging → intermediate → marts** layers.  
