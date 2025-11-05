# 🟢 dbt Variables — Key Concepts

## 🔹 Overview
Variables (`vars`) in dbt let you **parameterize your models, tests, or macros**, so you can easily control logic or filters without changing SQL code.  
They make projects flexible, reusable, and environment-aware.

---

## 🔹 Defining Variables

### Option 1 — In `dbt_project.yml`
```yaml
vars:
  cutoff_date: '2024-01-01'
  region: 'Canada'
```

### Option 2 — In the Command Line
```bash
dbt run --vars '{"cutoff_date": "2024-01-01"}'
```

### Option 3 — In a Macro or Model (Inline)
```sql
{% set threshold = var('sales_threshold', 1000) %}
```  

---

## 🔹 Using Variables in Models
Use variables inside SQL with `{{ var('name') }}` syntax:
```sql
select *
from {{ ref('stg_sales') }}
where order_date >= '{{ var("cutoff_date") }}'
```

---

## 🔹 Default Values
To prevent errors when a variable isn’t defined, add a default value:
```sql
{{ var("region", "DefaultRegion") }}
```

---

## 🔹 Common Use Cases
- Dynamic filters (e.g., cutoff date or region)  
- Environment toggles (e.g., dev vs prod)  
- Configuring thresholds for data quality checks  
- Parameterizing macros for reusable logic  

---

## 🔹 Summary
✅ Add flexibility and reusability  
⚙️ Define in `dbt_project.yml`, CLI, or inline Jinja  
🧠 Use `{{ var('name', 'default') }}` to avoid errors  
📦 Great for filters, thresholds, or environment logic  
