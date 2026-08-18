# 🟣 dbt Jinja — Key Concepts

**Jinja** is a templating language that makes SQL dynamic in dbt. It allows you to use variables, conditions, loops, filters, and macros inside SQL models — helping avoid repetition and keeping code modular and flexible.

Jinja is processed by dbt **before** the SQL is sent to the data warehouse. dbt evaluates the Jinja code and produces pure SQL that can then be executed by the warehouse.

---

## 🔹 Syntax Summary

* `{% %}` → run Jinja statements such as setting variables, conditions, and loops
* `{{ }}` → evaluate an expression and output its result
* `{# #}` → add Jinja comments, which do not appear in the compiled SQL

**Example:**

```sql id="u8o6gf"
{% set threshold = 100 %}

select *
from {{ ref('stg__sales') }}
where amount > {{ threshold }}

{# Only include rows above threshold #}
```

This could compile to:

```sql id="hdcghy"
select *
from analytics.staging.stg__sales
where amount > 100
```

### Using Literals Inside `{{ }}`

Jinja expressions inside `{{ }}` can contain **literal values** directly. A literal is a fixed value written directly in the code rather than stored in a variable.

For example:

```sql id="ckhwwz"
{{ 10 }}
{{ 'hello' }}
{{ true }}
{{ [1, 2, 3] }}
```

These represent different types of Jinja literals:

* `10` → number
* `'hello'` → string
* `true` → boolean
* `[1, 2, 3]` → list

Jinja can also evaluate expressions containing literals:

```sql id="4xoy0q"
{{ 10 + 5 }}
```

The result is:

```text id="25eyps"
15
```

For example:

```sql id="a2i6di"
select {{ 10 + 5 }} as result
```

compiles to:

```sql id="ybcaw3"
select 15 as result
```

### How `{{ }}` Is Rendered

Everything inside `{{ }}` is evaluated by Jinja, and the **result is rendered as text into the generated SQL**.

For example:

```sql id="gyf3g8"
select {{ 10 }}
```

compiles to:

```sql id="5hzvch"
select 10
```

Although Jinja renders its output into the SQL text, this does **not** mean the value automatically becomes a SQL string.

The final SQL determines how the warehouse interprets the generated text.

For example:

```sql id="d8eql1"
select {{ 10 }}
```

becomes:

```sql id="gwdjuh"
select 10
```

Here, `10` is interpreted by SQL as a number.

If we write:

```sql id="8wunqc"
select '{{ 10 }}'
```

it compiles to:

```sql id="yl6guj"
select '10'
```

Now SQL interprets the value as a string because quotes were included around the Jinja expression.

> **Important:** `{{ }}` evaluates an expression and inserts its rendered result into the generated SQL. It does not automatically add SQL quotes or convert the result into a SQL string literal.

Literals are also commonly passed as arguments to Jinja and dbt functions.

For example:

```sql id="kjbbwp"
{{ ref('stg__customers') }}
```

Here, `'stg__customers'` is a **Jinja string literal** passed as an argument to `ref()`.

Similarly:

```sql id="4ubnmj"
{{ var('cutoff_date') }}
```

Here, `'cutoff_date'` is a string literal representing the name of the dbt variable.

---

## 🔹 Jinja Features

### 1. Variables

Variables can be created using `set`.

```sql id="ygq5oa"
{% set threshold = 100 %}

select *
from {{ ref('stg__sales') }}
where amount > {{ threshold }}
```

Here, `threshold` is set to `100`, and:

```sql id="7ce7pz"
{{ threshold }}
```

outputs its value into the generated SQL.

The Jinja statement:

```sql id="fbj3s5"
{% set threshold = 100 %}
```

does not appear in the compiled SQL. It is evaluated by Jinja before the SQL is sent to the warehouse.

---

### 2. Conditions

Jinja allows conditional logic using `if`.

```sql id="stlxua"
{% if target.name == 'dev' %}
    limit 100
{% endif %}
```

This can be useful when different SQL should be generated depending on the environment or another condition.

If `target.name` is `dev`, Jinja generates:

```sql id="ueiqfj"
limit 100
```

If the condition is false, that piece of SQL is not generated.

The `if` statement itself does not appear in the compiled SQL.

---

### 3. Loops

Jinja `for` loops can be used to generate repetitive SQL dynamically.

```sql id="bgt30g"
{% for column in ['revenue', 'cost', 'profit'] %}
    sum({{ column }}) as total_{{ column }}
    {% if not loop.last %},{% endif %}
{% endfor %}
```

Jinja evaluates the loop before the SQL is executed.

The compiled SQL could look like:

```sql id="a0b4aa"
sum(revenue) as total_revenue,
sum(cost) as total_cost,
sum(profit) as total_profit
```

The `for` loop itself does **not** exist in the compiled SQL. It is used by Jinja to generate SQL.

---

### 4. Filters and the Pipe Operator `|`

Jinja provides **filters** that can transform or format values.

The `|` symbol is the **pipe operator**. It passes the value on its left to the filter on its right.

For example:

```sql id="4jupxv"
{% set columns = ['id', 'name', 'email'] %}

select {{ columns | join(', ') }}
from {{ ref('stg__users') }}
```

Here:

* `columns` contains a list of column names.
* `|` passes the list to the `join` filter.
* `join(', ')` combines the values using a comma and a space.

Therefore:

```sql id="u42o2p"
{{ columns | join(', ') }}
```

renders as:

```sql id="vx8stn"
id, name, email
```

The complete Jinja code could compile to:

```sql id="gttzy7"
select id, name, email
from analytics.staging.stg__users
```

Conceptually:

```text id="r8cvun"
columns → | → join(', ') → "id, name, email"
```

> **Note:** `join` here is a **Jinja filter** and is unrelated to a SQL `JOIN`.

Other Jinja filters include:

* `upper`
* `lower`
* `replace`
* `length`

---

### 5. Whitespace Control

Jinja statements themselves do not appear in the compiled SQL, but the **whitespace around the Jinja blocks can remain**.

Whitespace includes:

* Spaces
* Blank lines
* Line breaks

Jinja provides the `-` character to remove unwanted whitespace around Jinja blocks.

For example:

```sql id="xk6y0i"
{%- ... %}
```

removes whitespace **before** the Jinja block.

```sql id="ndfmp3"
{% ... -%}
```

removes whitespace **after** the Jinja block.

The same syntax can be used with expressions:

```sql id="st0x89"
{{- ... }}
{{ ... -}}
```

For example, without whitespace control:

```sql id="i7qk8v"
select
{% for column in ['id', 'name', 'email'] %}
    {{ column }}{% if not loop.last %},{% endif %}
{% endfor %}
from {{ ref('stg__users') }}
```

The `for`, `if`, and `endfor` statements disappear when Jinja generates the SQL, but line breaks and spaces surrounding those blocks can remain.

Using `-` allows us to remove unwanted whitespace:

```sql id="k4ntsg"
select
{%- for column in ['id', 'name', 'email'] %}
    {{ column }}{% if not loop.last %},{% endif %}
{%- endfor %}
from {{ ref('stg__users') }}
```

Whitespace control affects the **formatting of the generated SQL**, not the SQL logic or query result.

> **Important:** The `-` does not change what a loop, condition, or expression does. It only controls the whitespace that remains when Jinja generates the SQL.

---

### 6. Macros

Macros allow reusable pieces of Jinja logic to be defined once and called from different models.

For example:

```sql id="3fc4py"
{{ cents_to_dollars('amount') }}
```

Instead of repeating the same SQL logic in multiple models, the logic can be placed inside a macro and reused.

Macros are especially useful for:

* Avoiding repeated SQL
* Standardizing transformations
* Creating reusable business logic
* Generating SQL dynamically

---

## 🔹 Common dbt Uses

### Using `ref()`

`ref()` is used to reference another model within a dbt project.

```sql id="qgn8g4"
select *
from {{ ref('stg__customers') }}
```

dbt resolves `ref()` to the appropriate database, schema, and relation name.

For example, it could compile to something similar to:

```sql id="x73z7c"
select *
from analytics.staging.stg__customers
```

Using `ref()` also creates a dependency between models, allowing dbt to:

* Determine the correct model build order
* Build the project DAG and lineage
* Resolve relations correctly across environments

This is why models should generally be referenced using `ref()` rather than hard-coding their database and schema names.

---

### Using `source()`

`source()` is used to reference a source defined in dbt.

```sql id="47cp2j"
select *
from {{ source('airbnb_raw', 'listings') }}
```

dbt resolves the source definition into the appropriate database, schema, and table name when compiling the model.

---

### Using `var()`

Project variables can be accessed using `var()`.

```sql id="oy4t7a"
where order_date >= '{{ var("cutoff_date") }}'
```

`var()` retrieves the value of a dbt variable so values do not always need to be hard-coded into models.

Notice that the Jinja expression is surrounded by SQL quotes:

```sql id="3g8r41"
'{{ var("cutoff_date") }}'
```

If the variable contains:

```text id="a9y77m"
2026-08-01
```

the generated SQL would contain:

```sql id="72wp09"
where order_date >= '2026-08-01'
```

The quotes are part of the SQL, not something automatically added by `{{ }}`.

---
## 🔹 Jinja Functions for dbt

dbt provides additional **Jinja functions, variables, and context objects** that can be used inside dbt projects.

Some commonly used ones are:

- `target`
- `this`
- `log()`
- `var()`
- `env_var()`
- `adapter`

### `target`

`target` gives us access to information about the **active connection configuration that dbt is using to connect to the data platform**.

This information comes from the active target/output configured in `profiles.yml`.


#### Platform-Independent Attributes

These attributes are available across dbt platforms/adapters:

* `target.profile_name` → name of the dbt profile being used
* `target.name` → name of the active target, such as `dev` or `prod`
* `target.schema` → schema configured for the active target
* `target.type` → type of data platform/adapter, such as `snowflake`, `bigquery`, or `postgres`
* `target.threads` → number of threads dbt can use to execute work in parallel

For example:

```sql
{{ target.name }}
```

could render as:

```text
dev
```

`target` is especially useful when we want dbt to behave differently depending on the active environment:

```sql
{% if target.name == 'dev' %}
    limit 100
{% endif %}
```

If the active target is `dev`, the generated SQL includes:

```sql
limit 100
```

> **Note:** `target` is a Jinja context object rather than a function. It provides information about the active dbt connection/target configuration.

### `this`

`this` represents the **current model's database relation**.

It allows us to refer to the model that dbt is currently building without hard-coding its database, schema, or table name. We should use `this` instead of `ref()` because the model is trying to reference **itself**.

For example:

```jinja
{{ this }}
```

If the current model is `fct__orders`, it could render as:

```sql
AIRBNB_MART.ANALYTICS.FCT__ORDERS
```

`this` can also provide access to individual parts of the current relation:

```jinja
{{ this.database }}
{{ this.schema }}
{{ this.identifier }}
```

For example:

* `this.database` → database where the current model is being built
* `this.schema` → schema where the current model is being built
* `this.identifier` → current model's relation name

#### Common Use: Incremental Models

`this` is particularly useful in **incremental models**, where we may need to query the existing version of the model.

```sql
select *
from {{ ref('stg__orders') }}

{% if is_incremental() %}

where updated_at > (
    select max(updated_at)
    from {{ this }}
)

{% endif %}
```

If the current model is `fct__orders`, `{{ this }}` could compile to:

```sql
AIRBNB_MART.ANALYTICS.FCT__ORDERS
```

This allows the model to compare incoming records with the data that already exists in the target table.

We should use `this` here instead of `ref()` because the model is trying to reference **itself**.

For example, this would be a problem:

```jinja
{{ ref('fct__orders') }}
```

inside the `fct__orders` model itself.

Using `ref()` would create a dependency from the model back to itself, resulting in a **circular dependency** in the dbt DAG.

`this` avoids that problem because it refers directly to the current model's target relation without creating a dbt model dependency.

> **Note:** `this` is a dbt Jinja **context object**, not technically a function. It dynamically refers to the relation for the current model.



---

## 🔹 Summary

Jinja allows dbt to **generate SQL dynamically before execution**.

* `{% %}` → execute Jinja statements and logic
* `{{ }}` → evaluate an expression and output its result into the generated text
* `{# #}` → add Jinja comments
* Literals → fixed values such as strings, numbers, booleans, and lists
* `set` → define variables
* `if` → add conditional logic
* `for` → generate repetitive SQL
* `|` → apply a Jinja filter to a value
* `-` → control whitespace around Jinja blocks
* Macros → create reusable logic
* `ref()` → reference dbt models and establish dependencies
* `source()` → reference dbt sources
* `var()` → access dbt variables

### Key Idea

**Jinja generates text (usually SQL); the data warehouse executes and interprets the resulting SQL.**

Anything output through `{{ }}` is rendered into the generated SQL text, but this does **not** automatically make it a SQL string. Whether the generated value is interpreted as a string, number, identifier, or other SQL element depends on the SQL that Jinja generates.
