
# 🟢 dbt Data Tests — Quick Notes

## 🔹 What are Data Tests?

Data tests validate that your data meets expected quality rules. Tests should be automated, efficient, targeted, and informative, making it easier to identify the root cause when data quality issues occur.

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
Four dbt out-of-the-box tests:  

 - **`not_null`**: Ensures a column does not contain NULL values.
 - **`unique`**: Ensures every value in a column is unique.
 - **`accepted_values`**: Ensures values belong to a predefined list.
 - **`relationships`**: Validates referential integrity between two models.

```yaml
models:

  - name: stg_host_verifications
    columns:

      - name: verification_method
        data_tests:
          - accepted_values:
              arguments:
                values: ['email', 'phone', 'work_email', 'government_id']
              config:
                store_failures: true

  - name: fct_reviews
    columns:

      - name: reviewer_sk
        data_tests:
          - relationships:
              arguments:
                to: ref('dim_reviewer')
                field: reviewer_sk
```
---

# 🔹 Singular Tests

Singular tests are custom SQL files stored in the `tests/` folder.

They are used when built-in generic tests cannot express a business rule.

Example folder:

```text
tests/
└── superhosts_without_verification.sql
```

Example:

```sql
select *
from {{ ref('dim_host') }}
where host_is_superhost = true
  and has_gov_id = false
```

If this query returns any rows, the test fails.

### Reusing Macros in Singular Tests

Although singular tests are SQL files, they can call reusable macros.

This helps avoid duplicating complex SQL across multiple business rule tests.

Example:

**Macro:**

```sql
{% macro invalid_dates(model) %}

select *
from {{ model }}
where start_date > end_date

{% endmacro %}
```

**Singular Test:**

```sql
-- tests/invalid_dates.sql

{{ invalid_dates(ref('dim_customer')) }}
```

### This approach keeps singular tests simple while reusing the SQL logic through macros.  
---

## 🔹 Running Data Tests

### Running Tests

Run all tests:

```bash
dbtf test
```

Run tests for a specific model:

```bash
dbtf test --select dim_listing
```

Run tests for a model and all of its downstream models:

```bash
dbtf test --select dim_listing+
```

Run tests for a model and all of its upstream dependencies:

```bash
dbtf test --select +fct_reviews
```

Run tests for a model, including both upstream and downstream dependencies:

```bash
dbtf test --select +dim_listing+
```

Run tests for models with a specific tag:

```bash
dbtf test --select tag:reviews
```

Run tests based on model materialization:

```bash
dbtf test --select config.materialized:table
```

Run tests for models in a specific folder:

```bash
dbtf test --select path:models/marts/reviews
```

Run tests while excluding specific models:

```bash
dbtf test --exclude dim_date
```

Combine `--select` and `--exclude`:

```bash
dbtf test --select path:models/marts --exclude tag:experimental
```

Run tests using a predefined YAML selector:

```bash
dbtf test --selector nightly_tests
```

The selector must first be defined in `selectors.yml`. For example:

```yaml
selectors:
  - name: nightly_tests
    definition:
      method: tag
      value: nightly
```

Use deferred resolution so that unselected upstream dependencies can be resolved from a previous dbt state:

```bash
dbtf test --select fct_reviews --defer --state path/to/state
```

`--defer` is commonly used when you want to test a subset of models while referencing existing versions of dependencies that were not built in the current environment.

Run the entire project, including models, snapshots, seeds, and tests:

```bash
dbtf build
```


---

# 🔹 Test Configuration


Generic tests support several useful configuration options that control how tests are evaluated and how failures are handled.

- **Severity**

severity controls whether a failed test is treated as an error or a warning.
```yaml
columns:
  - name: listing_id
    data_tests:
      - unique:
          config:
            severity: warn
```
Possible values:

error
warn
Error If

- **error_if** defines the condition that determines when the number of failing records should result in an error.
```yaml
columns:
  - name: price
    data_tests:
      - not_null:
          config:
            error_if: "> 10"
```
In this example, the test produces an error when more than 10 records fail the test.

- **Warn If**

warn_if defines the condition that determines when the number of failing records should result in a warning.
```yaml
columns:
  - name: price
    data_tests:
      - not_null:
          config:
            warn_if: "> 0"
```
In this example, the test produces a warning when more than 0 records meet the warning condition.

Using Thresholds

error_if and warn_if can be used together to define different thresholds for test failures.
```yaml
columns:
  - name: price
    data_tests:
      - not_null:
          config:
            error_if: "> 10"
            warn_if: "> 0"
```
For example:

0 failures → Pass
1–10 failures → Warning
More than 10 failures → Error

This allows a small number of data quality issues to generate a warning while larger numbers of failures are treated as errors.
---

# 🔹 View Test Result

After tests run, results can be reviewed in several places depending on the level of detail needed.

## Terminal Output

The terminal displays a summary of each test result, including whether the test:

* **Passed**
* **Failed**
* Produced a **warning**

It also shows the number of failures detected by the test.

---

## dbt Debug Log

More detailed information about test execution can be found in the dbt log file, we can see the sql code ran also the error.

```text id="vlttvi"
logs/dbt.log
```

The debug log contains detailed execution information and can be useful when investigating test failures or unexpected behavior.

---

## Store and View Failing Records

By default, dbt reports test failures but does not permanently store the records that failed.

`store_failures` can be enabled in different ways depending on the desired scope.

### Test Level — Model YAML

Configure `store_failures` for an individual test:

```yaml id="s55myq"
columns:
  - name: listing_id
    data_tests:
      - unique:
          config:
            store_failures: true
```

This stores failing records for that specific test.

### Project Level — `dbt_project.yml`

`store_failures` can also be configured more broadly in `dbt_project.yml`:

```yaml id="4j8u5y"
data_tests:
  +store_failures: true
```

This can be used when failure records should be stored consistently across tests in the project.

### Command Line

Failure storage can also be enabled for a particular test execution:

```bash id="cy35vj"
dbtf test --store-failures
```

### How Stored Failures Work

When `store_failures` is enabled, dbt creates a **separate relation for each test** containing the records that failed that test.

For example, a uniqueness test on `listing_id` would have its own stored-failure relation containing the duplicate records identified by that test.

When the test runs again, the stored-failure relation is **replaced with the latest test results**. Therefore, it represents the failures from the latest execution rather than maintaining a history of failures across test runs.

Because of this, `store_failures` is primarily useful for **troubleshooting and investigating current data quality issues**.

> **Note:** `store_failures` should not be treated as a historical data quality monitoring solution. If historical test results or failure trends need to be tracked over time, a separate persistence or observability solution should be implemented.

### Summary

* **Terminal** → quick test status and failure count
* **`logs/dbt.log`** → detailed execution and debugging information
* **`store_failures`** → actual records that failed each test
* **Historical monitoring** → requires a separate solution to persist test results over time


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

The dbt community provides reusable packages and tools that extend dbt's built-in testing and data quality capabilities.

Popular and useful packages include:

* **dbt-utils** — commonly used utility macros and generic tests
* **dbt-expectations** — additional data quality tests inspired by Great Expectations
* **dbt-meta-testing** — helps validate that required tests and metadata are defined on dbt resources
* **dbt-data-quality** — provides additional functionality for implementing and managing data quality checks
* **dbt-audit-helper** — helps compare datasets and validate results, particularly useful when refactoring or migrating models
* **dbt-project-evaluator** — evaluates a dbt project against recommended modeling and project-structure practices

### Development and CI Tools

Some useful open-source tools are available through GitHub and can complement dbt testing:

* **dbt-checkpoint** — provides pre-commit hooks for validating dbt projects and enforcing development standards before changes are committed
* **dbt-coverage** — helps measure test and documentation coverage across a dbt project

### Explore More Packages

Additional community packages can be discovered through the **dbt Package Hub**:

`hub.getdbt.com`

The Package Hub can be used to explore available packages, review their documentation, and find packages for testing, data quality, auditing, monitoring, and other reusable dbt functionality.

Additional dbt-related tools that are not distributed through the Package Hub can also be found on GitHub.


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

1. **Prefer generic tests whenever possible**
   Use reusable generic tests for common data quality rules such as uniqueness, nullability, relationships, and accepted values. Use singular tests only for specific business rules or one-off validation logic that cannot be expressed cleanly with a generic test.

2. **Check open-source packages before building custom tests**
   Before creating your own generic test, review existing dbt community packages and GitHub tools. 

3. **Build tests as you build models**
    Follow test-driven development, testing should be part of model development, not something added afterward.

4. **Test source data as well as transformed models**
   Apply tests to important source fields so upstream data quality issues can be identified early.

   Source tests can often use `severity: warn` when an upstream issue should be surfaced without blocking the pipeline. Warnings can then be communicated to the upstream data owner while allowing downstream processing to continue when appropriate.

5. **Test important business keys**
   Primary and business keys should have appropriate tests. In most cases:

   * Use both `unique` and `not_null` for primary keys
   * Use `relationships` to validate foreign-key relationships
   * Use `accepted_values` for controlled categories, statuses, and lookup values

6. **Keep tests focused and meaningful**
   Each test should validate a clear data quality expectation and provide useful information when it fails. Avoid adding tests simply to increase test count. It has costs.

7. **Use failure thresholds when appropriate**
   Use `warn_if` and `error_if` when a small number of failures can be tolerated but larger numbers should cause the pipeline to fail.

8. **Store failed records when troubleshooting**
   Use `store_failures` when the actual failing records need to be investigated. Stored failures are useful for debugging but should not be treated as a historical data quality monitoring solution.

9. **Limit the test scope when appropriate**
   Use the `where` configuration to test only relevant subsets of large datasets when full-table testing is unnecessary. This can reduce execution time and warehouse compute costs.

10. **Run tests as part of deployment and CI workflows**
    Tests should run regularly and be integrated into the development and deployment process so data quality issues are detected before changes reach downstream consumers.


---

# 🔹 Summary

- ✅ Data tests validate data quality.
- 🧠 A data test is simply SQL that returns failing rows.
- 📄 Generic tests are reusable and defined in YAML.
- 💻 Singular tests are custom SQL files.
- 🚀 Run tests using `dbtf test`.
- 🔨 `dbtf build` runs models, snapshots, seeds, and tests.
- ⚙️ Configure tests with `severity`, `store_failures`, and `where`.
- 📦 Community packages such as **dbt-utils** and **dbt-expectations** provide many additional reusable tests.
