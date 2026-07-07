
# 🟣 dbt Snapshots — Quick Notes

## 🔹 What is a Snapshot?

A **snapshot** tracks historical changes to records over time by implementing **Slowly Changing Dimension (SCD) Type 2**.

Unlike a normal model that keeps only the latest version of a record, a snapshot preserves previous versions whenever tracked data changes.

---

## 🔹 When Should You Use a Snapshot?

- What was a customer's address last month?
- What department did an employee belong to last year?
- When did a product's price change?

Snapshots preserve historical values while keeping the current version available.

---

## 🔹 Typical Snapshot Workflow

```text
Raw Source
      │
      ▼
Staging Model (optional but recommended)
      │
      ▼
Snapshot
      │
      ▼
Historical Table (SCD Type 2)
```

> Although snapshots can reference a source directly, it is generally recommended to snapshot a staging model with minimal transformations.

---

## 🔹 Snapshot File

Snapshots are defined in **YAML** files (recommended).

```text
snapshots/
└── customer_snapshot.yml
```

---

## 🔹 Snapshot Template

```yaml
snapshots:
  - name: scd_sales__customers
    relation: "{{ ref('stg_customers') }}"
    config:
      strategy: timestamp
      unique_key: customer_id
      updated_at: updated_at
```

---

## 🔹 Naming Convention

```text
scd_<schema>__<table>
```

Example:

```text
scd_sales__customers
```

---

## 🔹 Snapshot Strategies

### Timestamp Strategy

Uses a timestamp column to determine whether a record has changed.

```yaml
config:
  strategy: timestamp
  updated_at: updated_at
```

### Check Strategy

Compares one or more columns to detect changes.

```yaml
config:
  strategy: check
  check_cols:
    - city
    - province
```

---

## 🔹 Running Snapshots

```bash
dbt snapshot
```

---

## 🔹 What Does dbt Create?

Running a snapshot creates a physical table and automatically adds:

- `dbt_scd_id`
- `dbt_updated_at`
- `dbt_valid_from`
- `dbt_valid_to`

---

## 🔹 Example

When a customer's city changes from Vancouver to Burnaby, dbt expires the old row by setting `dbt_valid_to` and inserts a new row with the updated city and a new `dbt_valid_from` value.

---

## 🔹 Snapshot vs Incremental

| Snapshot | Incremental |
|----------|-------------|
| Tracks history | Loads new/changed data efficiently |
| Preserves previous versions | Keeps latest version |
| Implements SCD Type 2 | Improves load performance |

---

## 🔹 Best Practices

- Snapshot raw or lightly transformed data.
- Prefer snapshotting a staging model.
- Use `ref()` or `source()` for lineage.
- Include as many business columns as possible from the beginning, we can not add column later.
- Keep transformations minimal.
- Avoid joins whenever possible.
- Prefer the `timestamp` strategy when a reliable `updated_at` column exists.
- Create a view if you need an `IsCurrent` or `Status` column.

---


