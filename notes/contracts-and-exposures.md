# 🟤 dbt Model Contracts & Exposures — Notes

## 🔹 Overview

Both features protect the **boundary between dbt and the people or tools that use its models**, such as a Power BI dashboard.

- A **model contract** locks the *shape* of a model: its column names and data types.
- An **exposure** describes *who uses* a model outside dbt, so it shows up in lineage.

They don't replace data tests:

| Feature | Checks | Question it answers |
| --- | --- | --- |
| Data tests | The **values** (unique, not null, accepted values) | Is the data correct? |
| Model contract | The **structure** (columns and types) | Does the model still look the way consumers expect? |
| Exposure | Nothing — it's documentation | Who depends on this model? |

> **Dota 2 project note:** In the Dota 2 project, Power BI connects directly to the mart tables instead of to views. Contracts and an exposure give the same protection a view layer would: a stable interface for the report, and visible lineage.

---

## 1️⃣ Model Contracts

### What is a Contract?

A **contract** is a promise about a model's output: which columns it returns and their data types.

When contract enforcement is on, dbt checks the model's SQL against the declared columns **before** building the table. If a column is missing, extra, renamed, or has a different data type, the build fails and the existing table is left as it is.

Without a contract, a renamed column simply replaces the table, and the problem only appears when the dashboard refreshes and breaks.

### Where a Contract is Defined

Contracts are configured in the model's YAML properties file:

```yaml
models:
  - name: dim_hero
    description: One row per hero, plus an unknown member.
    config:
      contract:
        enforced: true

    columns:
      - name: hero_key
        data_type: varchar
        description: Surrogate key, hash of the hero ID.
      - name: hero_id
        data_type: number
        description: Hero ID from the OpenDota API.
      - name: hero_name
        data_type: varchar
      - name: primary_attribute
        data_type: varchar
      - name: attack_type
        data_type: varchar
      - name: created_at
        data_type: timestamp_ntz
      - name: last_updated_at
        data_type: timestamp_ntz
```

### Rules to Remember

- **Every column must be listed**, each with a `data_type`. A column in the SQL but not in the YAML (or the other way round) fails the build.
- **Use the warehouse's type names**, for example Snowflake's `varchar`, `number`, `boolean`, `timestamp_ntz`.
- The **column order** in the YAML doesn't have to match the SQL.
- Contracts work on **table, view, and incremental** models. They are **not supported on ephemeral models**, since there's nothing in the warehouse to check.
- For **incremental** models, set `on_schema_change` to `append_new_columns` or `fail`.

### Constraints

A contracted model can also declare **constraints** on columns, such as `not_null` or `primary_key`:

```yaml
      - name: hero_key
        data_type: varchar
        constraints:
          - type: not_null
          - type: primary_key
```

Whether a constraint is **enforced** depends on the warehouse:

| Constraint | Snowflake |
| --- | --- |
| `not_null` | ✅ Enforced |
| `primary_key`, `unique`, `foreign_key` | ⚠️ Created as metadata only, not enforced |

> **Note:** Because Snowflake doesn't enforce primary keys or uniqueness, keep the `unique` and `not_null` **data tests** on keys even when you add constraints.

### Contract vs. Data Test Failure

```text
Column renamed in dim_hero.sql:  hero_name  →  name

Without contract:  dim_hero rebuilt  →  Power BI refresh fails later
With contract:     dbt build fails   →  dim_hero unchanged, report keeps working
```

### When to Use Contracts

- On **marts read by a BI tool or another team** (the models that are a public interface)
- Not needed on staging or intermediate models, which only dbt reads

---

## 2️⃣ Exposures

### What is an Exposure?

An **exposure** is a YAML entry describing something **outside dbt** that uses dbt models: a dashboard, a notebook, an ML model, or an application.

It doesn't build or run anything. It adds the consumer to the project as the **last node in the lineage graph**.

### Where an Exposure is Defined

Exposures go in any YAML file under `models/`, for example `models/exposures.yml`:

```yaml
exposures:
  - name: dota2_dashboard
    label: Dota 2 Player & Hero Dashboard
    type: dashboard
    maturity: medium
    url: https://app.powerbi.com/...
    description: >
      Two-page Power BI report: players and engagement,
      heroes and balance.

    depends_on:
      - ref('fact_matches')
      - ref('fact_match_players')
      - ref('dim_hero')
      - ref('dim_player')

    owner:
      name: Sahar Latifi
```

### Main Properties

| Property | Required | Meaning |
| --- | --- | --- |
| `name` | ✅ | Unique name, snake_case |
| `type` | ✅ | `dashboard`, `notebook`, `analysis`, `ml`, or `application` |
| `owner` | ✅ | `name` and/or `email` of the person responsible |
| `depends_on` | Recommended | The models and sources it reads, using `ref()` / `source()` |
| `label` | Optional | Friendly name shown in the docs |
| `maturity` | Optional | `high`, `medium`, or `low` |
| `url` | Optional | Link to the dashboard |
| `description` | Optional | What it is and who uses it |

### What an Exposure Gives Us

- **Lineage:** the dashboard appears in `dbt docs` as the final node, so anyone can see which marts feed it.
- **Impact analysis:** before changing a model, we can see which dashboards depend on it.
- **Selection:** build or test everything a dashboard needs with one command:

```bash
dbt build --select +exposure:dota2_dashboard
```

The `+` before the exposure selects all its upstream models.

---

## 🔹 Using Them Together

```text
staging → marts (contracts enforced) → exposure: Power BI dashboard
```

- The **contract** guarantees the marts keep the columns and types the dashboard expects.
- The **exposure** documents that the dashboard depends on those marts.
- **Data tests** confirm the values inside them are correct.

---

## 🔹 Summary

- 📜 A contract locks a model's column names and data types.
- 🛑 If the SQL doesn't match the contract, `dbt build` fails before replacing the table.
- 📝 Every column must be declared with a `data_type`.
- ⚠️ On Snowflake, only `not_null` constraints are enforced, so keep data tests on keys.
- 📊 An exposure documents a dashboard or app that uses dbt models.
- 🔗 Exposures appear in lineage and can be selected with `+exposure:<name>`.
- 🎯 Use contracts on marts that BI tools read, and one exposure per dashboard.
