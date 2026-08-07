# 🟣 dbt Snapshots — Quick Notes

## 🔹 What is a Snapshot?

A **snapshot** tracks historical changes to records over time by implementing **Slowly Changing Dimension (SCD) Type 2 (SCD2)**.

- Snapshots can reference either a `source()` or a `ref()`.
- Keep transformations to a minimum. If data cleansing or business logic is required, perform it **before** the snapshot (for example, in a staging model). Snapshot is not idempotent.    


---

## 🔹 Querying Snapshot Data

Snapshots can be queried in different ways depending on the business requirement.

### Current (Active) Record

To retrieve the latest version of each record, filter for the current record:

```sql
WHERE dbt_valid_to IS NULL
```

This is commonly used when building current-state dimension or fact tables.

> **Note:** Using only the current record may not accurately represent historical events because attribute values may have changed since those events occurred.

### Record at a Point in Time

To retrieve the version that was valid at a specific point in time, filter using the snapshot validity period:

```sql
event_date >= dbt_valid_from
AND event_date < COALESCE(dbt_valid_to, '9999-12-31')
```

This approach returns the version that was valid when the event occurred and is generally the most accurate way to join historical data to a snapshot.

## 🔹 Snapshot Frequency Considerations

A snapshot only captures the state of the source **at the time it runs**. It cannot detect changes that occur and are reversed between snapshot executions.

For example:

```text
9:00 AM   Customer status = Active
11:00 AM  Customer status = Suspended
2:00 PM   Customer status = Active
6:00 PM   Snapshot runs
```

The snapshot records only **Active**. The temporary **Suspended** state is never captured.

### Best Practices

- Choose a snapshot frequency that aligns with how frequently the source data changes.
- Consider the business importance of capturing every intermediate change.
- Understand the refresh frequency and update patterns of the source system.
- When joining snapshots to other tables, remember that some intermediate changes may never have been captured.

> **Note:** A snapshot is **not** a change data capture (CDC) solution. It records the state of the data only at the time the snapshot runs. If the source changes more frequently than the snapshot schedule, intermediate changes may be lost.
>
> 
---

## 🔹 Snapshot File

Snapshots are defined in **YAML** files.

```text
snapshots/
└── customer_snapshot.yml
```

---

## 🔹 Snapshot Template

```yaml
snapshots:
  - name: scd_sales__customers
    relation: "{{ source('sales', 'customers') }}"
    config:
      strategy: timestamp
      unique_key: customer_id
      updated_at: updated_at
```

> `relation` can reference either a `source()` or a `ref()`.

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

dbt provides two built-in snapshot strategies:

1. `timestamp`
2. `check`

### Timestamp Strategy

The timestamp strategy uses a date-time column to determine whether a record has changed.

```yaml
config:
  strategy: timestamp
  unique_key: customer_id
  updated_at: updated_at
```

The `timestamp` strategy is recommended when the source has a reliable `updated_at` column.

The `updated_at` value must change whenever any tracked attribute in the record changes. If the timestamp is not updated correctly, dbt may not detect the change.

> **Recommended approach:** Use the `timestamp` strategy whenever a reliable update timestamp is available.

---

### Check Strategy

The check strategy compares selected values between the current source record and the existing snapshot record.

Use the `check` strategy when the source does not have a reliable `updated_at` column.

There are three common approaches:

1. Check all columns.
2. Check a specific list of columns.
3. Check a generated hash column.



### Approach 1: Check All Columns

Use `check_cols: all` to compare all source columns.

```yaml
config:
  strategy: check
  unique_key: customer_id
  check_cols: all
```

#### When to use it

This approach may be suitable when there is no reliable `updated_at` column, and the source contains only a small number of columns.  

ℹ️ Always look at the query dbt generates in ***compile*** or ***run***. As the number of columns grows, dbt must compare more values. This can result in a larger and more complex query.


### Approach 2: Check a Column List

List only the columns whose changes should create a new historical version. dbt checks whether any of the listed columns have changed.

```yaml
config:
  strategy: check
  unique_key: customer_id
  check_cols:
    - city
    - province
    - customer_status
```

#### Advantages

* The generated comparison logic is usually simpler than checking every column.
* Only meaningful business changes create new snapshot versions.
* Unimportant source changes can be excluded.

#### Maintenance requirement

This approach requires manual maintenance.

When a new column is added and its history must be tracked, the developer must add it to `check_cols`.

```yaml
check_cols:
  - city
  - province
  - customer_status
  - customer_segment
```

If the new column is not added, dbt will not detect changes to that column.

> This approach reduces comparison complexity but requires developers to maintain the column list.


### Approach 3: Check a Hash Column

Create a hash value from all the business columns that should be tracked, then configure the snapshot to check the hash column.

For example, create the hash in the source query or an upstream model:

```sql
md5(
    concat_ws(
        '||',
        coalesce(city, ''),
        coalesce(province, ''),
        coalesce(customer_status, '')
    )
) as record_hash
```

Then configure the snapshot:

```yaml
config:
  strategy: check
  unique_key: customer_id
  check_cols:
    - record_hash
```

When one of the included business columns changes, the generated hash also changes. dbt detects the new hash and creates a new snapshot version.

#### Advantages

* dbt compares one hash column instead of many individual columns.
* The snapshot configuration remains simple.
* It can reduce the complexity of the snapshot comparison query.
* It can improve performance when many columns must be compared.
* It makes the set of historically tracked attributes explicit in the hash logic.

#### Maintenance requirement

The hash expression must be updated when a new business column should be tracked.

The hash should also handle:

* `NULL` values consistently.
* Data-type conversions consistently.
* Column order consistently.
* Delimiters carefully to avoid ambiguous combinations.

> **Recommended check approach:** When many columns must be monitored and no reliable timestamp exists, checking a carefully generated hash column can provide simpler comparison logic and better performance.

> The `timestamp` strategy is generally preferred because it handles column additions more easily and requires less ongoing maintenance. The `check` strategy is useful when the source does not provide a reliable update timestamp.

```

---

## 🔹 Running Snapshots

```bash
dbtf snapshot
```

---

## 🔹 What Does dbt Create?

Running a snapshot creates or updates a physical snapshot table and automatically adds the following metadata columns:

| Column | Purpose |
|--------|---------|
| `dbt_scd_id` | Unique identifier for each version of a record |
| `dbt_updated_at` | Source `updated_at` value (timestamp strategy) |
| `dbt_valid_from` | When this version became valid |
| `dbt_valid_to` | When this version expired (`NULL` = current version) |
| `dbt_is_deleted` | Indicates whether the record has been hard deleted from the source (`TRUE` = deleted, `FALSE` = not deleted). Only available when hard delete tracking is enabled. This column will not be created by default |


---

## 🔹 Example

When a customer's city changes from Vancouver to Burnaby, dbt closes the previous version by populating `dbt_valid_to` and inserts a new current version with an updated `dbt_valid_from`.

---

## 🔹 Hard Delete Handling

By default, dbt snapshots assume records are **updated**, not physically deleted. If a record is removed from the source, dbt will not detect the deletion unless hard delete tracking is configured.

### Enable Hard Delete Tracking

```yaml
config:
  strategy: timestamp
  unique_key: customer_id
  updated_at: updated_at
  hard_deletes: ignore | invalidate | new_record
```
---
## 🔹 Schema Changes

> **TODO:** Add notes about how dbt snapshots handle schema changes (e.g., added, removed, renamed, or modified columns).
---

## 🔹 Best Practices

- Snapshot raw or lightly transformed data.
- Use `ref()` or `source()` for lineage.
- The `unique_key` should uniquely identify a business entity and should never change.
- Include all business columns you may want to track from the beginning. New columns can be added later, but historical values for those columns will not exist for previously captured versions.
- Keep snapshots focused on preserving history rather than performing business transformations. Since snapshots are **not idempotent**, changes to transformation logic cannot be safely reapplied to historical records.
- Avoid joins whenever possible.
- Prefer the `timestamp` strategy when a reliable `updated_at` column exists.
- Ensure the `updated_at` column changes whenever any tracked attribute changes.
- Create a view if you need an `IsCurrent` or `Status` column.
