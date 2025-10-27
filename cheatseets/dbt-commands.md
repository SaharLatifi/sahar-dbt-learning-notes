# ⚙️ dbt CLI Commands Cheat Sheet

Below are the most commonly used dbt commands — organized by purpose.

---

## 🏗 Project Setup
- `dbtf init` — initialize a new dbt project  
- `dbt deps` — install dependencies from `packages.yml`  
- `dbtf debug` — test database connection and project setup  
- `dbtf --version` — show dbt version and installed adapters

---

## 🚀 Build & Run Models
- `dbtf run` — builds all models  
- `dbtf run --select my_model` — runs a specific model  
- `dbtf run --select tag:my_tag` — runs all models with the specified tag.  
- `dbt run --models +my_model` — run a model and its upstream dependencies  
- `dbt run --target prod` — run models using the prod target from `profiles.yml`  
- `dbt build` — build models, run tests, and snapshots together  
- `dbt build --select state:modified` — build only modified models (Slim CI)  

---

## 🧪 Testing & Documentation
- `dbt test` — run all tests  
- `dbt test --select my_model` — run tests for a specific model  
- `dbt docs generate` — generate documentation site  
- `dbt docs serve` — host the documentation locally  
- `dbt source freshness` — check data freshness of sources  

---

## ⏳ Snapshots & Seeds
- `dbt snapshot` — run snapshots defined in your project  
- `dbt seed` — load CSV files from the `seeds/` folder into your warehouse  
- `dbt seed --full-refresh` — reload all seeds, replacing existing tables  

---

## 🧹 Maintenance & Utilities
- `dbt clean` — delete the `target/` and `dbt_packages/` folders  
- `dbt compile` — compile all models to SQL without running them  
- `dbt parse` — validate and parse project structure without compiling  
- `dbt run-operation <macro>` — execute a macro manually (for maintenance tasks)  
- `dbt list` — list models, tests, or snapshots in your project  
- `dbt ls --select my_model+` — list a model and all downstream dependencies  

---

## 🧠 Debugging & Environment
- `dbt --version` — show dbt version  
- `dbt debug` — test connection and environment variables  
- `dbt debug --config-dir` — show location of configuration directories  
- `dbt --help` — view all available commands and options  
- `dbt <command> --help` — view detailed help for a specific command  

---

## 💡 Tips
- Combine selectors:  
  `dbt run --select tag:daily+` → run all models tagged as `daily` and their children  
- Use `--full-refresh` on any run to rebuild tables completely  
- Use `state:modified` with Slim CI for incremental testing  

---

### ✅ Quick Reference
| Category | Command Example | Purpose |
|-----------|-----------------|----------|
| Setup | `dbt init`, `dbt deps` | Initialize project & install packages |
| Build | `dbt run`, `dbt build` | Build models |
| Test | `dbt test` | Run data tests |
| Docs | `dbt docs generate` | Generate docs |
| Seeds | `dbt seed` | Load CSVs |
| Snapshots | `dbt snapshot` | Track historical changes |
| Maintenance | `dbt clean`, `dbt compile` | Clean or compile project |
| Debug | `dbt debug` | Test setup |

---

💬 *Tip:* Run `dbt build` instead of `dbt run` — it’s an all-in-one command that runs **models + tests + snapshots** in dependency order.



