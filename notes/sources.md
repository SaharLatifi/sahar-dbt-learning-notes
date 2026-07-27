# 🟢 dbt Sources — Key Concepts

## 🔹 Overview
`source()` is a Jinja function in dbt used to reference **raw data tables** stored in your warehouse.  
It keeps your models **environment-agnostic** and helps dbt build **data lineage** between your sources and models.  
Instead of hardcoding table paths, you define sources once and **reuse** them everywhere. 

- We use ref for everything we created in dbt, and we use source for the source tables in raw database.   
- It's good to have one source for each data source.
- When we use source, we can see the source tables in lineage.
- We can define freshness for sources.
- Each source YAML file supports a single folder or group of folders in a model hierarchy.
- dbt does nit create source by default, and we need to create it ourselves.
- We use source to define the raw data, source freshness, description, and tests.

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
This helps monitor data pipelines and identify stale data early before running the pipeline. This helps with the cost optimization. We are asking dbt to run the models, where the source data has been updated. **However it's not a replacement for monitoring data ingestion process.**    

We can define freshness at database level or table level. We also can override a data base level freshness by having more granular setting for a specific table.

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
        filter: <where-condition>
```
With filter we can limit the amount of data scanned -> usefull for improving performance. This only applies to source freshness query and not anything else in the source YML file.
We can also move this code after schema to check freshness for all the tables defined in that schema.   

### 🧠 Explanation
- **`loaded_at_field`** → column representing the last update timestamp  
- **`warn_after`** → dbt raises a warning if data is older than 1 day  
- **`error_after`** → dbt fails the freshness test if older than 2 days  

To run freshness checks:
```bash
dbt source freshness
```


dbt does not automatically run source freshness as part of dbt build or run.

This feature allow us to make a data-aware approach.    
We can instruct dbt to depend on sources that have been refreshed since their last successful run. Make your workflow more event-based by running:
``` bash
dbtf build --select source_status: fresher+
``` 

---  

### Complete Source Properties Syntax
``` yaml
sources:
  - name: <string> # required
    description: <markdown_string>
    database: <database_name>
    schema: <schema_name>
    loader: <string>

    # requires v1.1+
    config:
      <source_config>: <config_value>
      freshness:
      # changed to config in v1.10
      loaded_at_field: <column_name>
        warn_after:
          count: <positive_integer>
          period: minute | hour | day
        error_after:
          count: <positive_integer>
          period: minute | hour | day
        filter: <where-condition>
      meta: {<dictionary>} # changed to config in v1.10
      tags: [<string>] # changed to config in v1.10

    # deprecated in v1.10
    overrides: <string>

    quoting:
      database: true | false
      schema: true | false
      identifier: true | false

    tables:
      - name: <string> #required
        description: <markdown_string>
        identifier: <table_name>
        data_tests:
          - <test>
          - ... # declare additional tests
        config:
          loaded_at_field: <column_name>
          meta: {<dictionary>}
          tags: [<string>]
          freshness:
            warn_after:
              count: <positive_integer>
              period: minute | hour | day
            error_after:
              count: <positive_integer>
              period: minute | hour | day
            filter: <where-condition>

        quoting:
          database: true | false
          schema: true | false
          identifier: true | false
        external: {<dictionary>}
        columns:
          - name: <column_name> # required
            description: <markdown_string>
            quote: true | false
            data_tests:
              - <test>
              - ... # declare additional tests
            config:
              meta: {<dictionary>}
              tags: [<string>]
          - name: ... # declare properties of additional columns

      - name: ... # declare properties of additional source tables

  - name: ... # declare properties of additional sources
```

    Link to dbt documentation:    
    https://docs.getdbt.com/reference/source-properties?version=2.0&name=v2

--- 
### Benefits
- Reusability: Define in a central location, reuse across multiple models.
- Maintainability: Keep all datasource location in one place. Seperate your data source location from the actual code.
- Lineage:  
---
## 🔹 Summary
⚙️ Define in `_source.yml` under `/models/staging/[schema_name]/`  
🧭 Use `{{ source('source_name', 'table_name') }}` in SQL  
📅 Add `freshness` checks using `loaded_at_field` + time thresholds  
📦 dbt resolves it automatically per environment  
🧠 Key for lineage tracking and freshness testing  


## 📘 Reference
👉 [Official dbt Documentation — Sources](https://docs.getdbt.com/docs/build/sources)
