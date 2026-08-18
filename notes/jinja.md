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

### `log()`

`log()` is a dbt Jinja function used to **write messages to dbt's logs**.

It is useful for debugging macros, checking variable values, and understanding what dbt is doing while Jinja code is running.

Basic syntax:

```jinja id="pxh9se"
{{ log("This is a log message", info=True) }}
```

The `info` parameter controls whether the message is also written to standard output:

* `info=False` → writes the message to the dbt log file. This is the default.
* `info=True` → also displays the message in the command-line output.

#### Logging Variables

`log()` is particularly useful when debugging Jinja variables.

```jinja id="0ug8mb"
{% set threshold = 100 %}

{{ log("Threshold: " ~ threshold, info=True) }}
```

The `~` operator is used in Jinja to **concatenate values as strings**.

The output could look similar to:

```text id="lnf6br"
Threshold: 100
```

We can also use `log()` with dbt context objects:

```jinja id="5cprmx"
{{ log("Current target: " ~ target.name, info=True) }}
{{ log("Current model: " ~ this, info=True) }}
```

> **Note:** `log()` is mainly useful for **debugging and visibility during dbt execution**. It does not generate SQL that is sent to the data warehouse.

Tip: To test log() without creating a model, add it inside a macro and execute the macro using dbt run-operation.

---
### `adapter`

`adapter` is a dbt Jinja **context object** that gives us access to functionality provided by the active **data platform adapter**.

The adapter is the layer dbt uses to interact with a specific data platform, such as Snowflake, BigQuery, or Postgres.

Conceptually:

```text
dbt / Jinja
     ↓
  adapter
     ↓
Data Platform
```

The active adapter depends on the platform configured in the dbt profile.

For example, when using Snowflake:

```yaml
type: snowflake
```

dbt uses the Snowflake adapter.

#### What Can We Use `adapter` For?

`adapter` provides methods that allow Jinja code to interact with or retrieve metadata about objects in the data platform.

For example:

```jinja
{{ adapter.get_relation(...) }}
```

can be used to find a relation in the database.

Another useful method is:

```jinja
{{ adapter.get_columns_in_relation(this) }}
```

which retrieves information about the columns in a relation.

#### `this` vs `adapter`

`this` and `adapter` have different purposes:

* `this` → represents the **current model/relation**
* `adapter` → provides functionality for **interacting with the data platform**

They can also be used together:

```jinja
{{ adapter.get_columns_in_relation(this) }}
```

Here:

1. `this` identifies the current relation.
2. `adapter` communicates with the data platform to retrieve information about its columns.

A simple way to remember the difference:

> **`this` = What relation am I working with?**
> **`adapter` = How can dbt interact with the data platform?**

#### When Do We Use `adapter`?

We usually do not need to use `adapter` directly in normal dbt models.

Functions and objects such as:

```jinja
{{ ref('stg__customers') }}
{{ source('raw', 'customers') }}
{{ this }}
```

already handle many common operations for us.

`adapter` becomes more useful when writing **advanced macros** that need to inspect or interact with the data platform, such as:

* Checking whether a relation exists
* Retrieving columns from a relation
* Retrieving database metadata
* Performing adapter-specific operations

> **Note:** `adapter` is not technically a function. It is a dbt Jinja **context object** that provides methods for interacting with the active data platform adapter.

---
### `var()`

`var()` is a dbt Jinja function used to **access project-level variables**.

#### `vars`, `var()`, and `set`

It is important to distinguish between these three:

* `vars:` → defines **project-level variables** in `dbt_project.yml`
* `--vars` → supplies or overrides project variables from the command line for a dbt command
* `var()` → **accesses/retrieves** a project-level variable inside Jinja
* `{% set %}` → defines a **local Jinja variable** inside a model, macro, or other Jinja context

A simple way to remember:

> **`vars:`**** defines → ****`var()`**** reads → ****`set`**** creates locally**

#### Defining Project Variables

Project variables can be defined under `vars:` in `dbt_project.yml` as **key-value pairs**:

```yaml
vars:
  minimum_amount: 100
  start_date: '2026-01-01'
```

Here:

* `minimum_amount` → key
* `100` → value
* `start_date` → key
* `'2026-01-01'` → value

We use the **key** with `var()` to retrieve its corresponding value:

```jinja
{{ var('minimum_amount') }}
```

This returns:

```text
100
```

For example:

```sql
select *
from {{ ref('stg__orders') }}
where amount > {{ var('minimum_amount') }}
```

#### Overriding a Variable from the Command Line

A project variable can be supplied or overridden for a dbt command using `--vars`:

```bash
dbt run --vars '{"minimum_amount": 200}'
```

The values passed through `--vars` are also provided as **key-value pairs**.

In this example:

* `minimum_amount` → key
* `200` → value

For this command:

```jinja
{{ var('minimum_amount') }}
```

returns `200` instead of the value defined in `dbt_project.yml`.

This allows us to change configuration **without changing the model code**.

#### Providing a Default Value

`var()` can also provide a default value:

```jinja
{{ var('minimum_amount', 100) }}
```

This means:

* If `minimum_amount` is provided → use its value
* If it is not provided → use `100`

####
---

### `env_var()`

`env_var()` is a dbt Jinja function used to **access environment variables** from the environment where dbt is running.

Environment variables allow values to be provided from **outside the dbt project**, rather than hard-coding them in project files.

Basic syntax:

```jinja
{{ env_var('VARIABLE_NAME') }}
```

For sensitive values such as credentials, dbt supports environment variables prefixed with:

```text
DBT_ENV_SECRET_
```

For example:

```text
DBT_ENV_SECRET_USER
DBT_ENV_SECRET_PASSWORD
```

We can access them using:

```jinja
{{ env_var('DBT_ENV_SECRET_USER') }}
{{ env_var('DBT_ENV_SECRET_PASSWORD') }}
```

#### Providing a Default Value

`env_var()` can also accept a default value:

```jinja
{{ env_var('DBT_ENV_SECRET', 'dev') }}
```

This means:

- If `DBT_ENV_SECRET` exists → use its value
- If `DBT_ENV_SECRET` does not exist → use `dev`

Defaults are generally more appropriate for **non-sensitive configuration** than for credentials.

#### Common Use: `profiles.yml`

Environment variables are commonly used in `profiles.yml` to avoid hard-coding credentials.

For example:

```yaml
my_project:
  target: dev

  outputs:
    dev:
      type: snowflake
      account: "{{ env_var('DBT_ACCOUNT') }}"
      user: "{{ env_var('DBT_ENV_SECRET_USER') }}"
      password: "{{ env_var('DBT_ENV_SECRET_PASSWORD') }}"
      database: analytics
      schema: dbt_dev
```

The actual credential values are stored outside `profiles.yml` and retrieved when dbt runs.

This prevents sensitive credentials from being hard-coded and potentially committed to Git.

#### Using Environment Variables in CI/CD

When dbt runs locally, environment variables can be defined in the **local operating system environment**, such as Windows or WSL.

When dbt runs through **GitHub Actions**, credentials should normally be stored in **GitHub Secrets**, not in local Windows environment variables and not directly in the Git repository.

For example, a GitHub Secret could be created as:

```text
DBT_ENV_SECRET_PASSWORD
```

The GitHub Actions workflow exposes the secret to the runner as an environment variable:

```yaml
env:
  DBT_ENV_SECRET_PASSWORD: ${{ secrets.DBT_ENV_SECRET_PASSWORD }}
```

Then dbt can access it through `profiles.yml`:

```yaml
password: "{{ env_var('DBT_ENV_SECRET_PASSWORD') }}"
```

The flow is:

```text
GitHub Secret
      ↓
GitHub Actions environment variable
      ↓
env_var()
      ↓
profiles.yml
      ↓
dbt connection
```

Therefore:

- **Local dbt run** → local OS environment variable → `env_var()`
- **GitHub Actions CI/CD** → GitHub Secret → runner environment variable → `env_var()`

> **Important:** GitHub Secrets may be configured for a repository or GitHub environment, but the secret value is **not stored in the Git repository or committed with the code**.

#### `var()` vs `env_var()`

Both functions retrieve values, but from different places:

- `var()` → retrieves a **dbt project variable**
- `env_var()` → retrieves an **environment variable from the environment where dbt is running**

> **Key Idea:** `var()` is for **dbt project configuration**, while `env_var()` is for values supplied by the **external runtime environment**. For sensitive values such as credentials, use the `DBT_ENV_SECRET_` prefix and keep the actual secret outside the dbt repository.

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
