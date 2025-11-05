# 🟢 dbt Packages — Quick Notes

## 🔹 Overview
Packages in dbt are like **plugins or libraries** — collections of pre-built macros, tests, and utilities that extend dbt’s core functionality.  
They help teams **avoid reinventing the wheel** by reusing community-developed or official components.

---

## 🔹 Community & Official Packages
- dbt has an active community that maintains many **open-source packages** on [hub.getdbt.com](https://hub.getdbt.com).  
- Popular examples include:
  - **dbt_utils** — the most commonly used package with useful macros like `surrogate_key`, `get_relations_by_prefix`, and `union_relations`.  
  - **data_spine** — helps generate standard dimensions such as `dim_date` and `dim_time` automatically.  
  - **dbt_expectations**, **dbt_audit_helper**, etc. for data quality and auditing.

---

## 🔹 Installing Packages
Packages are managed via a `packages.yml` file in your project root.

Example:
```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: [">=1.0.0", "<2.0.0"]
```

Then run:
```bash
dbt deps
```
This downloads the packages into your `/dbt_packages` folder.

---

## 🔹 dbt Cloud & Fabric Fusion
If you use **dbt Cloud** or **Microsoft Fabric (dbt Fusion)**, common packages like **dbt_utils** are often **pre-installed automatically**, so you can use them right away without adding them manually.

---

## 🔹 Summary
✅ Extend dbt with ready-made macros and tests  
📦 Managed through `packages.yml` + `dbt deps`  
⚙️ Community-maintained and version-controlled  
🧠 Commonly use `dbt_utils` for everyday macros  
💡 `data_spine` can quickly generate `dim_date` and similar dimensions  
🚀 Fabric Fusion users get key packages pre-installed  
