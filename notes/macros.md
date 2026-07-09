# 🟣 dbt Macros — Key Concepts

## 🔹 Overview
Macros in dbt are like **functions** — reusable blocks of logic written in Jinja.  
They help you **avoid repetition**, **simplify transformations**, and **make SQL more modular** across your models.  
Although most macros generate SQL, they can also contain **pure Jinja code** for non-SQL logic (e.g., loops, string manipulation, or conditional logic).

---

## 🔹 Why Use Macros
- Reuse the same logic across multiple models  
- Keep SQL DRY (Don’t Repeat Yourself)  
- Handle complex or dynamic logic in one place  
- Parameterize repetitive patterns (e.g., surrogate keys, conditional filters)  
- Run standalone Jinja logic or utility scripts without needing SQL  

---

## 🔹 Creating a Macro
Macros are stored inside the `/macros` folder in your dbt project.

**Example:**
```sql
-- File: macros/calc_discount.sql
{% macro calc_discount(price, rate) %}
    {{ price }} * (1 - {{ rate }})
{% endmacro %}
```

You can write **SQL-based** or **pure Jinja-based** macros — dbt will compile them accordingly.

---

## 🔹 Using a Macro
You can call a macro inside a model, test, or another macro:
```sql
select
    order_id,
    {{ calc_discount('price', 0.10) }} as discounted_price
from {{ ref('stg_orders') }}
```

You can also **run a macro directly from the command line** using `run-operation`:
```bash
dbt run-operation calc_discount
```

---

## 🔹 Passing Variables to Macros
Macros can take arguments like functions:
```sql
{% macro my_filter(column_name, threshold) %}
    {{ column_name }} > {{ threshold }}
{% endmacro %}

-- Usage:
select * from {{ ref('stg_sales') }}
where {{ my_filter('revenue', 1000) }}
```

---

## 🔹 Using `target`

The `target` object gives information about the active dbt environment.

It is useful when you want to check which environment dbt is running in, such as `dev`, `test`, or `prod`.

- `target` provides information about the active dbt environment.
- `target.name` is commonly used to check whether dbt is running in `dev`, `test`, or `prod`.
- `target.schema` and `target.database` show where dbt is building objects.
- `dbt run-operation` can be used to run utility macros directly from the command line.


### Step 1: Create the Macro File

Create a new file in the `macros` folder:

```text
macros/show_target_info.sql
```



### Step 2: Add the Macro

```sql
{% macro show_target_info() %}
    {{ log('Target Information', info=True) }}
    {{ log('Schema: ' ~ target.schema, info=True) }}
    {{ log('Database: ' ~ target.database, info=True) }}
    {{ log('Type: ' ~ target.type, info=True) }}
    {{ log('Threads: ' ~ target.threads, info=True) }}
    {{ log('Target Name: ' ~ target.name, info=True) }}
{% endmacro %}
```



### Step 3: Run the Macro

```bash
dbt run-operation show_target_info
```



### Step 4: What This Shows

This macro prints information about the active target, including:

| Property | Meaning |
|----------|---------|
| `target.schema` | Active schema |
| `target.database` | Active database |
| `target.type` | Adapter type, such as Snowflake, BigQuery, or Databricks |
| `target.threads` | Number of threads configured for dbt |
| `target.name` | Active target name, such as `dev` or `prod` |



### Step 5: Use `target.name` for Environment Logic

You can use `target.name` to apply different logic depending on the environment.

```sql
{% if target.name == 'prod' %}
    {{ log('Running in Production', info=True) }}
{% else %}
    {{ log('Running in Development or Test', info=True) }}
{% endif %}
```

---
## 🔹 Useful Built-in Macros
- `{{ ref('model_name') }}` — references another model  
- `{{ source('source_name', 'table_name') }}` — references a raw source  
- `{{ var('variable_name') }}` — retrieves a dbt variable  
- `{{ dbt_utils.surrogate_key(['col1', 'col2']) }}` — generates a unique key  

## 🔹 Customizing Schema Names with `generate_schema_name`

dbt uses the `generate_schema_name` macro to determine the final schema where models are created.

By overriding this macro, you can apply different schema naming rules for different environments, such as Development and Production.
- `generate_schema_name` determines the final schema used by dbt.
- The default behavior can be overridden by creating a macro with the same name.
- It is commonly used together with `target.name` to apply different schema naming rules for Development and Production environments.
- This helps keep development and production environments isolated while using the same dbt project.


### Step 1: Create the Macro File

Create the following file:

```text
macros/generate_schema_name.sql
```


### Step 2: Add the Macro

```sql
{% macro generate_schema_name(custom_schema_name, node) %}

    {% if target.name == 'prod' %}
        {{ custom_schema_name | trim }}
    {% else %}
        {{ target.schema }}_{{ custom_schema_name | trim }}
    {% endif %}

{% endmacro %}
```


### Step 3: How It Works

The macro receives two parameters:

| Parameter | Description |
|-----------|-------------|
| `custom_schema_name` | The schema defined in your model configuration or `dbt_project.yml` |
| `node` | The current dbt resource (model, seed, snapshot, etc.) |

The macro returns the schema where dbt will create the object.

### Step 4: Example

Assume the following configuration:

```yaml
models:
  my_project:
    marts:
      +schema: marts
```

If your active target is:

```yaml
target:
  name: dev
  schema: sahar
```

The final schema becomes:

```text
sahar_marts
```

If your active target is:

```yaml
target:
  name: prod
```

The final schema becomes:

```text
marts
```


### Why Customize `generate_schema_name`?

Common use cases include:

- Use different schema naming conventions for Development and Production.
- Prevent developers from writing to shared production schemas.
- Standardize schema names across projects.
- Automatically isolate each developer's objects.

---


## 🔹 Using `exceptions`

The `exceptions` object allows a macro to stop execution or display warnings when validation fails.

It is commonly used to ensure required inputs are provided before a macro continues.

Example:

```sql
{% macro validate_schema(schema_name) %}

    {% if schema_name is none %}
        {{ exceptions.raise_compiler_error("schema_name is required.") }}
    {% endif %}

{% endmacro %}
```

Use `raise_compiler_error()` to stop execution with an error, or `warn()` to display a warning without stopping the run.
---

## 🔹 Using `run_query`

The `run_query()` function allows a macro to execute a SQL query against the data warehouse and use the returned results in Jinja.

It is useful for metadata queries, validations, dynamic SQL generation, and administrative tasks.

Example:

```sql
{% macro get_row_count(table_name) %}

    {% set results = run_query(
        "select count(*) from " ~ table_name
    ) %}

    {% if execute %}
        {% set row_count = results.columns[0].values()[0] %}
        {{ log("Row Count: " ~ row_count, info=True) }}
    {% endif %}

{% endmacro %}
```

Run the macro:

```bash
dbt run-operation get_row_count --args '{"table_name":"MY_DATABASE.MY_SCHEMA.CUSTOMERS"}'
```

### Common Use Cases

- Query metadata from the data warehouse.
- Check whether tables or schemas exist.
- Validate data before executing SQL.
- Generate dynamic SQL based on query results.

  ---
  


## 🔹 Summary
✅ Makes SQL reusable and modular  
⚙️ Defined in `/macros` folder  
🧠 Written using Jinja syntax (`{% macro %}` … `{% endmacro %}`)  
📦 Useful for logic, filters, and automation across models  
💡 Can be run standalone via `dbt run-operation`  
🧩 Supports both SQL and pure Jinja logic inside macros  
