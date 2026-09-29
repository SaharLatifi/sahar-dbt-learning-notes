# dbt Incremental Models — Learning Note

## 1. What Is an Incremental Model?

An incremental model is a dbt model that does not rebuild the entire target table every time it runs.

- On the first run, dbt creates and loads the entire table.
- On subsequent runs, dbt processes only the records that meet the incremental filter.
- Depending on the incremental strategy, dbt can append new rows, update existing rows, or do both.

The main idea is:

> Keep the existing table and process only the data that needs to be added or changed.

## 2. Why Use an Incremental Model?

The main reason to use an incremental model is performance and scalability.

If a table has 100 million rows and only 10,000 new or changed records arrive, rebuilding all 100 million rows every time is inefficient. An incremental model allows the pipeline to process only the relevant portion of the data.

### Benefits

- Reduces processing time
- Reduces warehouse compute usage
- Reduces unnecessary data movement
- Makes pipelines more scalable as data grows
- Allows frequent refreshes without rebuilding large tables

## 3. Basic Incremental Configuration

```sql
{{ config(
    materialized='incremental'
) }}
```

A common configuration also includes a `unique_key`:

```sql
{{ config(
    materialized='incremental',
    unique_key='match_id'
) }}
```

### `materialized='incremental'`

Tells dbt that the model should be built incrementally instead of recreating the complete table on every run.

### `unique_key`

Identifies the record that should be considered unique. It is especially important for strategies such as `merge`.

## 4. First Run vs. Subsequent Runs

### First Run

On the first run, there is no existing target table, so the full dataset is processed.

```text
Source → All records → Incremental model → Target table created
```

### Subsequent Runs

On later runs, the incremental logic determines which records need to be processed.

```text
Source → Incremental filter → New/changed records → Existing target table
```

## 5. `is_incremental()`

The most important dbt function for controlling incremental logic is:

```sql
is_incremental()
```

It returns `true` when dbt is running the model incrementally.

Example:

```sql
SELECT *
FROM {{ ref('stg_matches') }}

{% if is_incremental() %}
WHERE start_time > (
    SELECT MAX(start_time)
    FROM {{ this }}
)
{% endif %}
```

On the first run, `is_incremental()` is false, so the filter is not applied. On subsequent runs, it is true and the filter is applied.

## 6. `{{ this }}`

`{{ this }}` refers to the current dbt model's target relation.

For example:

```sql
SELECT MAX(start_time)
FROM {{ this }}
```

means to look at the existing target table for this model and find the latest `start_time`.

## 7. Limiting the Data in an Incremental Model

The incremental filter determines how much data is processed during each incremental run.

Example:

```sql
{% if is_incremental() %}
WHERE start_time > (
    SELECT MAX(start_time)
    FROM {{ this }}
)
{% endif %}
```

This processes records newer than the latest record already stored.

Another approach is a fixed lookback window:

```sql
{% if is_incremental() %}
WHERE start_time >= DATEADD(day, -3, CURRENT_DATE)
{% endif %}
```

A lookback window can help handle late-arriving or corrected records.

## 8. Choosing the Incremental Filter

The filter should be based on how the source data behaves.

### Timestamp-based

```sql
WHERE updated_at > (
    SELECT MAX(updated_at)
    FROM {{ this }}
)
```

Useful when the source has a reliable `updated_at` column.

### Date-based lookback

```sql
WHERE event_date >= DATEADD(day, -3, CURRENT_DATE)
```

Useful when late-arriving data is possible.

### ID-based

```sql
WHERE id > (
    SELECT MAX(id)
    FROM {{ this }}
)
```

Useful when IDs are sequential and new records always have larger IDs.

The important question is:

> How can I reliably identify data that is new or has changed?

## 9. Incremental Strategies

The incremental strategy determines how dbt applies the selected records to the target table.

Common strategies include:

- Append
- Merge
- Delete + Insert
- Insert Overwrite
- Microbatch

The exact strategies available depend on the database adapter and configuration.

## 10. Append Strategy

The append approach adds new rows to the existing table.

It is appropriate when:

- Records are immutable
- Existing records do not need updates
- The source only produces new records

A key limitation is that append does not automatically update an existing record.

## 11. Merge Strategy

The `merge` strategy can both update existing records and insert new records. It typically relies on a `unique_key`.

Example:

```sql
{{ config(
    materialized='incremental',
    unique_key='match_id',
    incremental_strategy='merge'
) }}
```

Conceptually:

```text
Incoming record
      ↓
Does the unique key already exist?
      ↓
 Yes → UPDATE
 No  → INSERT
```

## 12. Delete + Insert

The delete + insert approach generally:

1. Identifies matching existing records
2. Deletes them
3. Inserts the incoming records

This can be useful when records need to be replaced rather than updated column by column.

## 13. Insert Overwrite

Insert overwrite replaces data in a defined portion of the target table, commonly a partition.

For example, if data is partitioned by date, the affected partition can be replaced instead of rebuilding the entire table.

## 14. Microbatch

Microbatch processing divides incremental data into smaller batches, often based on time intervals.

```text
Large incremental range
        ↓
 ┌──────┬──────┬──────┬──────┐
 │Batch1│Batch2│Batch3│Batch4│
 └──────┴──────┴──────┴──────┘
```

This can be useful for very large time-series datasets.

## 15. Incremental Models and `unique_key`

`unique_key` is especially important when records can change.

For example:

```sql
{{ config(
    materialized='incremental',
    unique_key='player_id'
) }}
```

The key should represent the grain of the model. If the model grain is one row per player per match, for example, `player_id` alone would not be a sufficient unique key.

## 16. Incremental Models for Facts and Dimensions

Incremental models are commonly useful for large fact tables because fact tables can grow quickly.

Examples:

```text
fact_matches
fact_player_stats
fact_transactions
fact_events
```

Dimensions require more consideration because dimension records can be updated.

Examples:

```text
dim_player
dim_team
dim_customer
```

If dimension attributes can change, a merge strategy or another appropriate update pattern may be required.

## 17. Full Refresh

An incremental model can still be rebuilt completely with a full refresh.

Normal run:

```text
Existing table + New/changed data → Incremental update
```

Full refresh:

```text
Source → Rebuild → Complete target table
```

A full refresh can be useful when:

- The model logic changes significantly
- Historical data needs to be corrected
- The incremental logic has changed
- The target table needs to be rebuilt
- The incremental process has produced incorrect results

## 18. Common Incremental Model Pattern

```sql
{{ config(
    materialized='incremental',
    unique_key='match_id',
    incremental_strategy='merge'
) }}

WITH source_data AS (

    SELECT
        match_id,
        start_time,
        game_mode,
        region
    FROM {{ ref('stg_matches') }}

    {% if is_incremental() %}

    WHERE start_time >= DATEADD(
        day,
        -3,
        CURRENT_DATE
    )

    {% endif %}

)

SELECT *
FROM source_data
```

The important pieces are:

```text
materialized='incremental'
        ↓
Tell dbt to use incremental materialization

unique_key
        ↓
Identify records

is_incremental()
        ↓
Apply special filtering only on incremental runs

incremental filter
        ↓
Limit the source data

incremental_strategy
        ↓
Determine how records are applied to the target
```

## 19. Common Mistakes

### Mistake 1: Assuming incremental materialization automatically filters the source

Setting:

```sql
materialized='incremental'
```

does not automatically mean that only new records are selected. You still need to define the incremental filtering logic when your model requires it.

### Mistake 2: Using the wrong `unique_key`

The `unique_key` must match the actual grain of the model.

If the model has one row per player per match, using only `player_id` as the unique key would incorrectly identify multiple records as the same record.

### Mistake 3: Using `>` without considering late-arriving data

A filter such as:

```sql
WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
```

can miss records that arrive late or have timestamps that fall within an already-processed period.

A lookback window can help address this situation.

### Mistake 4: Forgetting that incremental models depend on existing state

Incremental behavior depends on the existing target table. Test both:

- A full-refresh run
- A normal incremental run

This helps verify that the model works correctly when the target table is first created and when it already contains data.

## 20. Practical Mental Model

Think about an incremental model as three separate decisions:

### 1. What data should I process?

Controlled by the incremental filter:

```sql
{% if is_incremental() %}
...
{% endif %}
```

### 2. How do I identify the record?

Controlled by:

```sql
unique_key
```

### 3. How should the record be applied?

Controlled by:

```sql
incremental_strategy
```

So:

```text
                 Incremental Model
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
     What data?     Which row?    How apply?
          │             │             │
   is_incremental   unique_key   strategy
                                  │
                         append / merge /
                         delete+insert /
                         insert overwrite /
                         microbatch
```

## 21. Key Takeaways

- An incremental model avoids rebuilding the entire target table on every run.
- The first run normally processes the full dataset.
- Later runs process only the data selected by the incremental logic.
- `is_incremental()` lets you apply different logic on incremental runs.
- `{{ this }}` refers to the current target table.
- The incremental filter controls which source records are processed.
- A lookback window can help handle late-arriving or corrected data.
- `unique_key` identifies records when updates are required.
- The incremental strategy determines how selected records are written to the target.
- `merge` is useful when existing records can change.
- `append` is useful when records are immutable and only new rows arrive.
- Full refresh remains available when the entire model needs to be rebuilt.
