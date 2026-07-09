
       # 🔵 dbt Documentation — Quick Notes

## 🔹 What is dbt Documentation?

dbt can automatically generate documentation for your project.

The generated documentation includes:
- Models
- Sources
- Snapshots
- Seeds
- Columns
- Lineage (dependency graph)
- Metadata

The documentation is presented as an interactive website that helps developers and business users understand the project.

---

## 🔹 Where is Documentation Written?

Documentation is not written inside SQL models.

Instead, dbt documentation is typically written in YAML files.

A common approach is to create one YAML file for each model (or one YAML file per folder).

Example:

```text
models/
├── staging/
│   ├── stg_sales__customers.sql
│   ├── stg_sales__customers.yml
│
├── marts/
│   ├── dim_customer.sql
│   ├── dim_customer.yml
```

The YAML file typically has the same name as the model it documents.

Inside the YAML file you can define:
- descriptions
- tests
- metadata (meta)
- model configuration (config)

Example:

```yaml
models:
  - name: dim_customer
    description: Customer dimension.

    columns:
      - name: customer_id
        description: Unique customer identifier.
      - name: city
        description: Customer city.
```

---

## 🔹 Using `meta`

Example:

```yaml
models:
  - name: dim_customer
    meta:
      owner: Analytics Team
      contact: data-team@company.com
      business_domain: Sales
```

Common uses:
- Owner
- Contact
- Team
- Business domain
- Slack channel
- Jira ticket

---

## 🔹 Persisting Documentation to the Warehouse

Enable `persist_docs` to write model and column descriptions into the warehouse (when supported).

```yaml
models:
  my_project:
    +persist_docs:
      relation: true
      columns: true
```

---

## 🔹 Generating Documentation

```bash
dbt docs generate
```

Creates:
- manifest.json
- catalog.json
- index.html

---

## 🔹 Viewing Documentation

```bash
dbt docs serve
```

Starts a local documentation website with lineage, models, columns, sources, snapshots, seeds and metadata.

---

## 🔹 dbt Platform Catalog

The dbt Platform provides **Catalog**, an enhanced documentation experience with richer search, navigation, governance and collaboration features.

---

## 🔹 Best Practices

- Create one YAML file per model (or folder).
- Keep the YAML filename the same as the model name.
- Document every model.
- Document important columns.
- Use `meta` for ownership and governance.
- Enable `persist_docs` when supported.
- Keep documentation up to date.

---

## 🔹 Summary

- Documentation is written in YAML files.
- Typically create one YAML file per model.
- Use `description` for models and columns.
- Use `meta` for custom metadata.
- Enable `persist_docs` to write descriptions to the warehouse.
- Generate docs with `dbt docs generate`.
- View docs with `dbt docs serve`.
- dbt Platform Catalog provides a richer documentation experience.
