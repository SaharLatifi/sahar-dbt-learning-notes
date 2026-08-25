# 🟢 dbt Packages — Quick Notes

## 🔹 Overview

Packages in dbt are like **plugins or libraries**—collections of pre-built macros, tests, and utilities that extend dbt’s core functionality.

They help teams **avoid reinventing the wheel** by reusing community-developed or organization-developed components.

---

## 🔹 Popular Packages

dbt has an active community that maintains open-source packages on the **[dbt Package Hub](https://hub.getdbt.com)**.

Some popular packages include:

* **dbt_utils** – provides macros such as `generate_surrogate_key`, `union_relations`, `get_relations_by_prefix`, and `date_spine`.
   Its macros can also make your code more **portable across different database adapters**, reducing the need for database-specific SQL.
* **dbt_expectations** – inspired by the Python package Great Expectations, it provides many reusable data-quality tests, including:
  - Range checks – verify that numeric values fall within an expected range.
  - Table-shape checks – validate row counts, column counts, and table structure.
  - String-matching checks – verify that values match a pattern or regular expression.
  - Distribution checks – validate averages, medians, and other statistical properties.
  - Cross-table checks – compare values or row counts between different tables.

It helps implement more advanced data-quality checks without writing every generic test from scratch.
* **dbt_audit_helper** – helps compare tables, validate migrations, and audit data changes.
* **dbt_codegen** – generates repetitive dbt code automatically.
   Important macros include:
    - generate_source – generates source YAML from existing database tables.
    - generate_model_yaml – generates model YAML, including column names.
    - generate_base_model – generates a basic staging model from a source.
    - generate_model_import_ctes – generates import CTEs for referenced models.
    - generate_model_import_ctes – generates import CTEs for referenced models.
* **dbt_project_evaluator** – evaluates a dbt project against dbt lab recommended best practices. You can incorporate it in your CI pipeline.
  It can identify issues related to:
  - Model structure and naming
  - Documentation and testing
  - Model dependencies
  - Source usage
  - Project performance and maintainability
---

## 🔹 The `packages.yml` File

Packages are managed using a `packages.yml` file.

The file should be in the **same directory as `dbt_project.yml`**, usually the root directory of the dbt project.

```text
my_dbt_project/
├── dbt_project.yml
├── packages.yml
└── models/
```

---

## 🔹 Adding Packages

There are two main ways to add a package.

### 1. From the dbt Package Hub

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.3.0
```

### 2. From a Git Repository

```yaml
packages:
  - git: "https://github.com/dbt-labs/dbt-utils.git"
    revision: 1.3.0
```

The `revision` can be a Git tag, branch name, or commit hash.

---

## 🔹 Installing and Updating Packages

Run:

```bash
dbt deps
```

This installs or updates packages in the `dbt_packages/` directory, or in the directory configured by `packages-install-path` in `dbt_project.yml`.

Run `dbt deps` again whenever you add or change a package in `packages.yml`.

---

## 🔹 Package Versioning

### Exact Version

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.3.0
```

This keeps builds predictable and reproducible.

### Version Range

You can specify a version range to automatically receive compatible updates.

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: [">=1.3.0", "<1.4.0"]
```

This allows **patch updates**, such as `1.3.1` or `1.3.2`, but prevents upgrading to version `1.4.0`.

Use a controlled version range when you want bug fixes and patches without unexpected larger changes.

---

## 🔹 Package Sources

Packages can come from:

* dbt Package Hub
* Git repositories
* Local packages within your organization

---

## 🔹 Summary

* ✅ Packages provide reusable macros, tests, and utilities.
* 📦 Manage packages with `packages.yml`.
* 📁 Keep `packages.yml` beside `dbt_project.yml`.
* 🌐 Add packages through the dbt Package Hub or a Git URL.
* ⚙️ Install or update packages using `dbt deps`.
* 🔒 Use an exact version for maximum reproducibility.
* 🔄 Use a controlled version range to receive compatible patch updates.
* 📅 Use `date_spine` from `dbt_utils` to generate continuous date or timestamp ranges.
