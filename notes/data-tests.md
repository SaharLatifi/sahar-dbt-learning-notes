
# 🟢 dbt Data Tests — Quick Notes

## 🔹 What are Data Tests?

Data tests validate that your data meets expected quality rules.

A dbt data test is simply a **SQL query that returns failing records**.

- ✅ **0 rows returned** → Test **PASS**
- ❌ **1 or more rows returned** → Test **FAIL**

This makes data tests easy to understand and customize.

---

## 🔹 How Data Tests Work

```text
dbt test
      │
      ▼
Execute SQL
      │
      ▼
Rows Returned?
      │
 ┌────┴────┐
 │         │
0 Rows   >0 Rows
 │         │
PASS     FAIL
```

---

## 🔹 Types of Data Tests

dbt supports two types of data tests.

| Generic Tests | Singular Tests |
|---------------|----------------|
| Built-in reusable tests | Custom SQL tests |
| Defined in YAML | Defined in SQL files |
| Reusable across many models | Written for one specific validation |
| Best for common data quality rules | Best for business-specific validation rules |

---

## 🔹 When Should You Use Each Type?

### Generic Tests

Use generic tests for **common data quality validations** that can be reused across many models.

Examples:

- Primary keys are unique.
- Required columns are not null.
- Status values are valid.
- Foreign keys exist in the parent table.

---

### Singular Tests

Use singular tests for **business-specific validation rules** that cannot be expressed using built-in generic tests.

Examples:

- Every active customer must have at least one order.
- The invoice total must equal the sum of its line items.
- A discharged patient cannot have an admission date after the discharge date.
- A customer cannot have more than one active subscription.

---

# 🔹 Generic Tests

Generic tests are reusable test macros that are applied through a model's YAML file.

They are the most commonly used data tests in dbt.

---

### `not_null`: Ensures a column does not contain NULL values.
### `unique`: Ensures every value in a column is unique.
### `accepted_values`: Ensures values belong to a predefined list.
### `relationships`: Validates referential integrity between two models.

```yaml
models:
  - name: fct_all_ticket_sales
    columns:
      - name: payment_method
        data_tests:
          - accepted_values:
              arguments: # available in v1.10.5 and higher. Older versions can set the <argument_name> as the top-level property.
                values: ['Cash', 'Credit Card', 'Debit Card', 'Mobile Payment']
                config:
                  store_failures: true
      - name: customer_id
        data_tests:
          - relationships:
              arguments: # available in v1.10.5 and higher. Older versions can set the <argument_name> as the top-level property.
                to: ref('dim_customers')
                field: customer_id
```

---

# 🔹 Singular Tests

Singular tests are custom SQL files stored in the `tests/` folder.

They are used when built-in generic tests cannot express a business rule.

Example folder:

```text
tests/
└── customers_without_email.sql
```

Example:

```sql
select *
from {{ ref('dim_customer') }}
where email is null
```

If this query returns any rows, the test fails.

---

## 🔹 Running Data Tests

Run all tests:

```bash
dbt test
```

Run tests for a specific model:

```bash
dbt test --select dim_customer
```

Run the entire project (models, snapshots, seeds, and tests):

```bash
dbt build
```

---

# 🔹 Test Configuration

Generic tests support several useful configuration options.

---

## Severity

Controls whether a failed test produces an **error** or a **warning**.

```yaml
columns:
  - name: customer_id

    data_tests:
      - unique:
          config:
            severity: warn
```

Possible values:

- `error`
- `warn`

---

## Store Test Failures

Store failing rows in the data warehouse for debugging.

```yaml
columns:
  - name: customer_id

    data_tests:
      - unique:
          config:
            store_failures: true
```

Instead of only reporting that the test failed, dbt stores the failing rows in a table, making debugging much easier.

---

## Test a Subset of Data

Use `where` to test only part of the data.

```yaml
columns:
  - name: customer_id

    data_tests:
      - unique:
          config:
            where: "order_date >= current_date - 30"
```

This reduces execution time and warehouse compute costs.

---

# 🔹 Community Testing Packages

The dbt community provides many reusable testing packages.

Popular examples include:

- **dbt-utils**
- **dbt-expectations**

These packages provide many additional reusable tests beyond the built-in generic tests.

---

# 🔹 Cost Considerations

Data tests execute SQL against your data warehouse.

Testing very large tables may increase execution time and compute costs.

Best practices:

- Test only the data you need.
- Use `where` to limit large datasets.
- Avoid unnecessary full-table scans.
- Schedule expensive tests appropriately.

---

# 🔹 Best Practices

- Add generic tests to important business keys.
- Use both `unique` and `not_null` for primary keys.
- Use `relationships` to validate foreign keys.
- Use `accepted_values` for status or lookup columns.
- Use singular tests for business-specific validation rules.
- Store failed rows when debugging.
- Use `where` to reduce testing costs on large tables.
- Run tests regularly as part of your deployment pipeline.

---

# 🔹 Summary

- ✅ Data tests validate data quality.
- 🧠 A data test is simply SQL that returns failing rows.
- 📄 Generic tests are reusable and defined in YAML.
- 💻 Singular tests are custom SQL files.
- 🚀 Run tests using `dbt test`.
- 🔨 `dbt build` runs models, snapshots, seeds, and tests.
- ⚙️ Configure tests with `severity`, `store_failures`, and `where`.
- 📦 Community packages such as **dbt-utils** and **dbt-expectations** provide many additional reusable tests.
