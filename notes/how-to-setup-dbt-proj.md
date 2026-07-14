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
   - WSL: Allows VS code to open and work directly inside your WSL (Linux) environment.
   - Remote Explorer: Lets you view and manage remote environments, including WSL, making it easy to connect to and switch between them.

5. **Open VS Code in WSL**
   - Open VS Code
   - Press ^+shift+p
   - Run: WSL: Connect to WSL or WSL: Reopen folder in WSL
   - Open the project folder  

7. **Install dbt extension in VS Code**  
   (Optional but recommended for syntax highlighting and navigation)  

8. **Install dbt-fusion**  
   Run the official installer in WSL:  

   ```bash
   curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update
   ```

   Check version:  
   ```bash
   dbtf --version
   ```

9. **Initialize a new project**  
   ```bash
   mkdir ~/projects && cd ~/projects
   dbtf init --project-name my_dbt_project  -->> Config Snowflake connection
   cd my_dbt_project - you need to be in the dbt project root folder so the extension works properly
   run code . to open and reload the window
   ```

10. **Configure Snowflake profile**  
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

   

11. **Verify connection**  
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
