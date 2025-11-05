# 🟣 dbt Jinja — Key Concepts

**Jinja** is a templating language that makes SQL dynamic in dbt. It allows you to use variables, conditions, loops, and macros inside SQL models — helping avoid repetition and keeping your code modular and flexible.

Jinja is processed by dbt *before* the SQL is sent to your data warehouse. This means dbt replaces or evaluates all Jinja expressions (`{% %}`, `{{ }}`, `{# #}`) and produces pure SQL for execution.

---

## 🔹 Syntax Summary
- `{% %}` → run commands like setting variables, if conditions, or for loops  
- `{{ }}` → replace or output a value (e.g., model names, variables, expressions)  
- `{# #}` → add comments or notes (ignored during execution)

**Example:**
```sql
{% set threshold = 100 %}

select *
from {{ ref('stg_sales') }}
where amount > {{ threshold }}

{# Only include rows above threshold #}
```

---

## 🔹 Common Uses
- Reference other models → `{{ ref('stg_customers') }}`  
- Reference sources → `{{ source('airbnb_raw', 'listings') }}`  
- Use variables → `{{ var('cutoff_date') }}`  
- Create reusable logic:
  ```sql
  {% if target.name == 'dev' %}
      limit 100
  {% endif %}
  ```
- Set variables or perform calculations:
  ```sql
  {% set columns = ['id', 'name', 'email'] %}
  select {{ columns | join(', ') }} from {{ ref('users') }}
  ```

- Using `ref()`:  

    `ref()` is used to reference another model within your dbt project.  
    It automatically resolves the correct database, schema, and table name — ensuring models build in the right order and remain environment-agnostic. dbt can show the lineage only if we use ref function.

    ```sql
    select * from {{ ref('stg_customers') }}


---

## 🔹 Summary
✅ Makes SQL **dynamic and reusable**  
⚙️ `{% %}` for logic, `{{ }}` for values, `{# #}` for comments  
📦 Useful for loops, variables, and referencing models  
🧠 Ideal for automating and cleaning up repetitive SQL  
