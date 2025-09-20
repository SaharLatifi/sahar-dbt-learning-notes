## Profiles vs Project Configs

- `profiles.yml` defines **connection settings** (warehouse, user, schema).
- It is created in the **home directory** (`~/.dbt/`) — not inside a project.
- Reason: you may have multiple dbt projects but want to share the same profile.
- Best practice:
  - Keep `profiles.yml` in your home `~/.dbt/` folder.
  - Do not commit real credentials into Git.

