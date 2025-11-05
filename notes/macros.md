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

## 🔹 Useful Built-in Macros
- `{{ ref('model_name') }}` — references another model  
- `{{ source('source_name', 'table_name') }}` — references a raw source  
- `{{ var('variable_name') }}` — retrieves a dbt variable  
- `{{ dbt_utils.surrogate_key(['col1', 'col2']) }}` — generates a unique key  

---

## 🔹 Summary
✅ Makes SQL reusable and modular  
⚙️ Defined in `/macros` folder  
🧠 Written using Jinja syntax (`{% macro %}` … `{% endmacro %}`)  
📦 Useful for logic, filters, and automation across models  
💡 Can be run standalone via `dbt run-operation`  
🧩 Supports both SQL and pure Jinja logic inside macros  
