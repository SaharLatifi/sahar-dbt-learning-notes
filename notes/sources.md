# 🟢 dbt Sources — Key Concepts

## 🔹 Overview
`source()` is a Jinja function in dbt used to reference **raw data tables** stored in your warehouse.  
It keeps your models **environment-agnostic** and helps dbt build **data lineage** between your sources and models.  
Instead of hardcoding table paths, you define sources once and **reuse** them everywhere. 

We use ref for everything we created in dbt, and we use source for the source tables in raw database.   

When we use source, we can see the source tables in lineage.

---

## 🔹 Defining Sources
Sources are defined in a  `_source.yml` file within each staging model folder dbt project. We usually have one source.yml file for each schema. use _source.yml as name, it will stays on top. 

Example:
```yaml
version: 2

sources:
version: 2
- name: listings
  database: AIRBNB_RAW
  schema: listings        
  tables:
    - name: listings
    - name: neighbourhoods
    - name: reviews 
```

---

## 🔹 Using `source()` in Models
Once defined, we can reference them dynamically:
```sql
select *
from {{ source('listings', 'reviews') }}
```
source has two arguments, the source name and the table name. 
dbt automatically resolves this to the correct database and schema (e.g., `AIRBNB_RAW.listings.reviews`), depending on your environment.

---

## 🔹 Defining Freshness
We can define **freshness checks** inside the source configuration to ensure that raw tables are updated within expected time limits.  
This helps monitor data pipelines and identify stale data early before running the pipeline. This helps with the cost optimization.

Example:
```yaml
version: 2

sources:
  - name: listings
    schema: listings
    database: airbnb_raw
    tables:
      - name: listings
        description: "Airbnb property listings table"
        loaded_at_field: last_scraped    # column that indicates when data was last updated
        freshness:
          warn_after: {count: 1, period: day}
          error_after: {count: 2, period: day}
```

### 🧠 Explanation
- **`loaded_at_field`** → column representing the last update timestamp  
- **`warn_after`** → dbt raises a warning if data is older than 1 day  
- **`error_after`** → dbt fails the freshness test if older than 2 days  

To run freshness checks:
```bash
dbt source freshness
```


```

## 🔹 Summary
⚙️ Define in `_source.yml` under `/models/staging/[schema_name]/`  
🧭 Use `{{ source('source_name', 'table_name') }}` in SQL  
📅 Add `freshness` checks using `loaded_at_field` + time thresholds  
📦 dbt resolves it automatically per environment  
🧠 Key for lineage tracking and freshness testing  


## 📘 Reference
👉 [Official dbt Documentation — Sources](https://docs.getdbt.com/docs/build/sources)
