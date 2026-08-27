# 🟢 dbt Hooks — Notes

## 🔹 Overview

A **hook** is a SQL statement—or a list of SQL statements—that dbt executes at a specific point in its workflow.

Hooks allow us to run custom warehouse operations before or after dbt performs another action. They are useful when the required behavior is not available through dbt's standard configurations.

Unlike dbt models, hooks are not limited to `SELECT` statements. Depending on the data platform and the user's permissions, they can execute commands such as:

- `INSERT`, `UPDATE`, and `DELETE`
- `CREATE`, `ALTER`, and `DROP`
- `GRANT` and other permission commands
- Warehouse-specific maintenance or administrative commands

Hooks can also call macros that return executable SQL.

---

## 🔹 Common Use Cases

Hooks can be used to:

- Grant the reporting role access to an Airbnb mart model after it is created
- Insert Airbnb model-run information into an audit table
- Delete temporary Airbnb records before rebuilding a model
- Apply Snowflake-specific settings to Airbnb tables
- Perform maintenance on the Airbnb analytics schemas
- Record the start, completion, or result of an Airbnb dbt invocation

---

## 🔹 Hooks and Related Operations

There are two main categories of dbt lifecycle hooks, plus the related `run-operation` command:

1. **`Pre-hooks` and `post-hooks`**
2. **`on-run-start` and `on-run-end` hooks**
3. **`dbt run-operation`** — related to hooks, but technically a separate command

---

## 1️⃣ Pre-Hooks and Post-Hooks

Pre-hooks and post-hooks run immediately before or after an individual dbt resource is built.

They can be applied to:

- Models
- Seeds
- Snapshots

### Where Pre-Hooks and Post-Hooks Can Be Defined

Pre-hooks and post-hooks are dbt configurations. They can be defined in either:

- The SQL model file, using a `config()` block
- A YAML file, such as a model properties file or `dbt_project.yml`

A model can have both a pre-hook and a post-hook associated with it. The pre-hook runs before the model is built, and the post-hook runs after the model is built.

### Using Multiple SQL Statements

A hook is not limited to one SQL statement. We can provide a list of statements, and dbt executes them consecutively in the order in which they are defined.

In a SQL model:

```sql
{{ config(
    pre_hook=[
      "delete from airbnb_mart.audit.temp_listing_records",
      "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'started')"
    ],
    post_hook=[
      "grant select on {{ this }} to role airbnb_reporting_role",
      "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'completed')"
    ]
) }}

select *
from {{ ref('stg_listings') }}
```

In YAML, each SQL statement is a separate list item:

```yaml
models:
  - name: dim_listing
    config:
      post_hook:
        - "grant select on {{ this }} to role airbnb_reporting_role"
        - "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'completed')"
```

### Defining Both Hooks in a SQL Model

```sql
{{ config(
    pre_hook="insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'started')",
    post_hook="insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'completed')"
) }}

select *
from {{ ref('stg_listings') }}
```

### Defining Both Hooks in YAML

```yaml
models:
  - name: dim_listing
    config:
      pre_hook:
        - "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'started')"
      post_hook:
        - "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'completed')"
```

### Pre-Hook

A pre-hook runs **before** the selected model, seed, or snapshot is built.

```sql
{{ config(
    pre_hook="delete from airbnb_mart.audit.temp_listing_records"
) }}

select *
from {{ ref('stg_listings') }}
```

### Post-Hook

A post-hook runs **after** the selected model, seed, or snapshot is built successfully.

```sql
{{ config(
    post_hook="grant select on {{ this }} to role airbnb_reporting_role"
) }}

select *
from {{ ref('stg_listings') }}
```

Pre-hooks and post-hooks can be configured in:

- A resource's `config()` block
- A YAML properties file
- The `dbt_project.yml` file

### Should This Be a Model or a Hook?

Before creating a hook, ask:

> Could this logic be implemented as its own dbt model and referenced as an upstream dependency?

If the logic creates or transforms a dataset that other models depend on, it should usually be a model. This allows dbt to include it in the DAG, manage dependencies, test it, document it, and rebuild it predictably.

Use a hook when the operation is a side effect or warehouse action that does not naturally belong in the model DAG—for example, granting access to `dim_listing` or writing its execution status to an audit table.

### Transaction Behavior

Transaction behavior depends on the database adapter.

For adapters that support transactions in dbt, particularly PostgreSQL and Redshift, pre-hooks and post-hooks run inside the same transaction as the model by default.

This means:

- If the model fails, operations performed by its pre-hook can be rolled back.
- If the post-hook fails, the model transaction can also be rolled back.
- The model and its hooks are treated as one transactional unit.

Sometimes a hook must run outside the model transaction. For example, we may want to record that `dim_listing` started even if the model later fails.

For supported transactional adapters, a hook can be defined as a dictionary containing:

- `sql`: the SQL statement to execute
- `transaction`: whether the hook should run inside the model transaction

### SQL Model Configuration

```sql
{{ config(
    pre_hook={
      "sql": "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'started')",
      "transaction": False
    },
    post_hook={
      "sql": "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'completed')",
      "transaction": False
    }
) }}
```

### YAML Configuration

```yaml
models:
  airbnb_dbt:
    marts:
      +pre-hook:
        sql: "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'started')"
        transaction: false
      +post-hook:
        sql: "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'completed')"
        transaction: false
```

- `transaction: true` runs the hook inside the same transaction as the model. This is the default behavior on supported transactional adapters.
- `transaction: false` runs the hook outside the model transaction, so its changes are committed independently.

> **Airbnb project note:** The Airbnb project uses Snowflake. Do not use the `transaction` hook configuration with Snowflake. It is intended for adapters where dbt supports transactional hook behavior, particularly PostgreSQL and Redshift. The Airbnb examples above demonstrate the syntax, but this transaction setting should not be applied to the Snowflake project itself.


### Complete Hook Execution Order

For a dbt invocation, the overall order is:

1. `on-run-start` hooks run once at the beginning of the invocation.
2. Pre-hooks defined in dependent packages run.
3. Pre-hooks defined in the project's `dbt_project.yml` run.
4. Pre-hooks defined in the node's SQL `config()` block or YAML properties run.
5. The model, seed, or snapshot is built.
6. Post-hooks defined in dependent packages run.
7. Post-hooks defined in the project's `dbt_project.yml` run.
8. Post-hooks defined in the node's SQL `config()` block or YAML properties run.
9. `on-run-end` hooks run once at the end of the invocation.

Steps 2–8 describe the lifecycle of an individual resource. When dbt runs multiple resources, it still follows DAG dependencies and may execute independent resources concurrently using multiple threads.

Hooks are cumulative. If hooks are defined at multiple configuration levels, dbt executes all of them rather than allowing one level to replace another. Within the same level, hooks execute in the order in which they are listed.

For example:

```yaml
models:
  airbnb_dbt:
    marts:
      +pre-hook:
        - "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'project hook 1')"
        - "delete from airbnb_mart.audit.temp_listing_records"
```

```sql
{{ config(
    pre_hook=[
      "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'model hook 1')",
      "insert into airbnb_mart.audit.model_log (model_name, event) values ('dim_listing', 'model hook 2')"
    ]
) }}

select *
from {{ ref('stg_listings') }}
```

Assuming no package hooks apply, dbt executes this portion of the sequence in the following order:

1. Project hook 1
2. Delete temporary listing records
3. Model hook 1
4. Model hook 2

The same package → project → node precedence applies separately to post-hooks. `on-run-start` and `on-run-end` surround the complete invocation; they do not run once for every individual model.

---

## 2️⃣ `on-run-start` and `on-run-end`

These are project-level hooks that run at the beginning or end of a dbt invocation.

- **`on-run-start`** runs once at the beginning of the invocation.
- **`on-run-end`** runs once at the end of the invocation.

They apply to the following commands:

- `dbt build`
- `dbt compile`
- `dbt docs generate`
- `dbt run`
- `dbt seed`
- `dbt snapshot`
- `dbt test`

### Airbnb Run-Log Example

For the Airbnb project, we can use:

- `on-run-start` to create the log table if it does not already exist
- `on-run-end` to insert the result of every executed dbt resource into the log table

The logic is placed in macros, while `dbt_project.yml` calls those macros.

#### Step 1: Create the Log Table

Create `macros/create_airbnb_log_table.sql`:

```sql
{% macro create_airbnb_log_table(log_table_name) %}

    create table if not exists
        {{ target.database }}.{{ target.schema }}.{{ log_table_name }} (
            invocation_id varchar,
            node_unique_id varchar,
            node_name varchar,
            resource_type varchar,
            status varchar,
            execution_time_seconds float,
            run_started_at timestamp_tz,
            logged_at timestamp_tz
        )

{% endmacro %}
```

The macro receives the table name as an argument. `target.database` and `target.schema` come from the active dbt target.

#### Step 2: Insert the Run Results

Create `macros/log_airbnb_results.sql`:

```sql
{% macro log_airbnb_results(results, log_table_name) %}

    {% if execute and results | length > 0 %}

        insert into {{ target.database }}.{{ target.schema }}.{{ log_table_name }} (
            invocation_id,
            node_unique_id,
            node_name,
            resource_type,
            status,
            execution_time_seconds,
            run_started_at,
            logged_at
        )

        {% for result in results %}

            select
                '{{ invocation_id }}',
                '{{ result.node.unique_id }}',
                '{{ result.node.name }}',
                '{{ result.node.resource_type }}',
                '{{ result.status }}',
                {{ result.execution_time }},
                '{{ run_started_at }}'::timestamp_tz,
                current_timestamp()

            {% if not loop.last %} union all {% endif %}

        {% endfor %}

    {% endif %}

{% endmacro %}
```

`results` is available in the `on-run-end` context and contains one result object for every resource executed during the dbt invocation.

#### Step 3: Call the Macros from `dbt_project.yml`

```yaml
on-run-start: "{{ create_airbnb_log_table('DBT_RUN_LOG') }}"

on-run-end: "{{ log_airbnb_results(results, 'DBT_RUN_LOG') }}"
```

### Where the Log Values Come From

| Log column | dbt value | Meaning |
| --- | --- | --- |
| `invocation_id` | `invocation_id` | UUID generated once for the entire dbt command |
| `node_unique_id` | `result.node.unique_id` | Unique dbt identifier, such as `model.airbnb_dbt.dim_listing` |
| `node_name` | `result.node.name` | Resource name, such as `dim_listing` |
| `resource_type` | `result.node.resource_type` | Type such as model, test, seed, or snapshot |
| `status` | `result.status` | Execution result, such as success, error, or skipped |
| `execution_time_seconds` | `result.execution_time` | Time taken to execute the resource |
| `run_started_at` | `run_started_at` | Timestamp when the dbt invocation began |
| `logged_at` | `current_timestamp()` | Timestamp when the log row was inserted |

The combination of `invocation_id` and `node_unique_id` identifies a particular resource within a particular dbt invocation. One invocation can therefore have many log rows.

These hooks can contain SQL statements or call macros that return SQL.

---

## 3️⃣ `dbt run-operation`

`dbt run-operation` is **not a lifecycle hook**. It is a command used to invoke a macro manually and independently of a model run.

```bash
dbt run-operation grant_select --args '{role: airbnb_reporting_role}'
```

Arguments can be passed to the macro using `--args`:

```bash
dbt run-operation refresh_airbnb_permissions --args '{role: airbnb_reporting_role}'
```

It is useful for:

- Granting permissions
- Performing one-time maintenance
- Cleaning up old database objects
- Running administrative SQL
- Executing reusable operational macros on demand

In newer dbt versions, `run-operation` can also execute SQL directly with the `--sql` option:

```bash
dbt run-operation --sql "drop table if exists airbnb_mart.audit.old_listing_log"
```

---

## 🔹 Quick Comparison

| Mechanism | When It Runs | Scope | Trigger |
| --- | --- | --- | --- |
| `pre-hook` | Before a resource is built | Individual model, seed, or snapshot | Automatically during the relevant dbt command |
| `post-hook` | After a resource is built | Individual model, seed, or snapshot | Automatically during the relevant dbt command |
| `on-run-start` | At the beginning of an invocation | Entire dbt invocation | Automatically |
| `on-run-end` | At the end of an invocation | Entire dbt invocation | Automatically |
| `run-operation` | When explicitly requested | A selected macro or SQL operation | Manually from the command line |

---

## 🔹 Topics to Add Next

- Detailed pre-hook and post-hook configuration
- Defining hooks in `dbt_project.yml` versus resource files
- Using multiple hooks and understanding execution order
- Calling macros from hooks
- Using `{{ this }}`, `target`, `schemas`, and `results`
- Transaction behavior and adapter differences
- Detailed `on-run-start` and `on-run-end` examples
- Creating and executing `run-operation` macros
- Practical auditing, permission, and cleanup examples
- Hook limitations and best practices
