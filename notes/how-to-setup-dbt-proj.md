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

   #### Check if WSL is already installed

      ```bash
      wsl --status
      ```
4. **Install VS Code extensions**  
   - WSL  
   - Remote Explorer  

5. **Initialize WSL in VS Code**  
   - Open a *Remote Window*  
   - Connect to WSL  

6. **Install dbt extension in VS Code**  
   (Optional but recommended for syntax highlighting and navigation)  

7. **Install dbt-fusion**  
   Run the official installer in WSL:  

   ```bash
   curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update
   ```

   Check version:  
   ```bash
   dbtf --version
   ```

8. **Initialize a new project**  
   ```bash
   mkdir ~/projects && cd ~/projects
   dbtf init my_dbt_project  -->> Config Snowflake connection
   cd my_dbt_project
   ```

9. **Configure Snowflake profile**  
   To edit your dbt profile file in WSL:  
   ```bash
   nano ~/.dbt/profiles.yml
   Or locate profile.yml first and edit the file
   ( echo %USERPROFILE% )
   find ~ -name profiles.yml 2>/dev/null
   code ~/.dbt/profiles.yml


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
   ```

   

10. **Verify connection**  
   ```bash
   dbtf debug
   ```

   If everything is configured correctly, you should see:  
   ```
   All checks passed!
   ```

11. **First run**  
    ```bash
    dbtf build       # run models, tests, seeds, snapshots

    ```

---

✅
