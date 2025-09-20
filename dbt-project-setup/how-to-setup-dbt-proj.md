# Setup a dbt Project (Snowflake) — with dbt-fusion (WSL installer method)

This guide shows how to install **dbt-fusion** in WSL using the official installer script, and connect it to Snowflake.

---

## Why WSL?
- Windows Subsystem for Linux (WSL) lets you run Linux directly inside Windows, without needing a VM.  
- Most dbt tutorials, packages, and scripts assume a Linux/Mac environment.  
- Dependency management (like adapters, Python tools) is cleaner and less error-prone in Linux.  
- You can edit with VS Code on Windows while running dbt in Linux for a smooth workflow.  

---

## Steps

1. **Create Snowflake trial account**  
   [Sign up here](https://signup.snowflake.com/?trial=student)

2. **Install WSL**  
   Follow the [Microsoft instructions](https://learn.microsoft.com/en-us/windows/wsl/install)

3. **Install VS Code extensions**  
   - WSL  
   - Remote Explorer  

4. **Initialize WSL in VS Code**  
   - Open a *Remote Window*  
   - Connect to WSL  

5. **Install dbt extension in VS Code**  
   (Optional but recommended for syntax highlighting and navigation)  

6. **Install dbt-fusion**  
   Run the official installer in WSL:  

   ```bash
   curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update
   ```

   Check version:  
   ```bash
   dbt-fusion --version
   ```

7. **Initialize a new project**  
   ```bash
   mkdir ~/projects && cd ~/projects
   dbt-fusion init my_dbt_project
   cd my_dbt_project
   ```

8. **Configure Snowflake profile**  
   Edit your dbt profile file in WSL:  
   ```bash
   nano ~/.dbt/profiles.yml
   ```

   Example configuration:  
   ```yaml
   my_dbt_project:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: "<account>"
         user: "<username>"
         password: "<password>"
         role: "<role>"
         database: "<DATABASE>"
         warehouse: "<WAREHOUSE>"
         schema: DEV
         threads: 4
   ```

   Save with `CTRL+O`, press Enter, then `CTRL+X`.

9. **Verify connection**  
   ```bash
   dbt-fusion debug
   ```

   If everything is configured correctly, you should see:  
   ```
   All checks passed!
   ```

10. **First run**  
    ```bash
    dbt-fusion deps        # install packages (e.g., dbt-utils)
    dbt-fusion build       # run models, tests, seeds, snapshots
    dbt-fusion docs generate
    dbt-fusion docs serve  # open docs in your browser
    ```

---

✅ You now have dbt-fusion installed in WSL, connected to Snowflake, and ready to build your first project.
