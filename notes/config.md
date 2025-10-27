## Profiles vs Project Configs

- `profiles.yml` defines **connection settings** (warehouse, user, schema, etc.).
- It is created in the **home directory** (`~/.dbt/`) — not inside a project.
- Reason: you may have multiple dbt projects but want to share the same profile.
- Best practices:
  - Keep `profiles.yml` in your home `~/.dbt/` folder.
  - Do not commit real credentials into Git.

There are different ways to configure dbt:

## Config Levels in dbt

There are three main places where you can define configuration in dbt. Each has a different purpose and level of precedence.

---

### 1. Project Level (`dbt_project.yml`)

Sets default configurations for all models, or for models within a specific folder (e.g., all models in your `staging` folder are views).  
This is the **lowest level of precedence**.

```yml
models:
  your_project_name:
    staging:
      +materialized: view
    marts:
      +materialized: table
```

---

### 2. Model Level (`.sql` file)

This is the **most specific level**.  
A `config()` block at the top of a model’s `.sql` file will override any setting from the project level.  
For example, you can force one model to be a table even if the project default is a view.

```jinja
{{ config(materialized='view') }}
```

---

### 3. Model Metadata (`.yml` file)

These files are **not for model configuration**.  
They are used exclusively for **adding documentation** (descriptions) and **tests** to your models and their columns.  

```yml
version: 2

models:
  - name: my_model
    description: "This model contains customer order data"
    columns:
      - name: customer_id
        description: "Unique identifier for each customer"
        tests:
          - not_null
          - unique
```

⚠️ The **higher priority** for configs always comes from the **model `.sql` file**.  
YML files are strictly for **docs and testing**, not materialization or other configs.
