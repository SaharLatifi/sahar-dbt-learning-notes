# Setup a dbt Project (Snowflake) — with dbt-fusion (WSL installer method)

This guide shows how to install **dbt-fusion** in WSL using the official installer script, and connect it to Snowflake.

Windows Subsystem for Linux (WSL) lets you run Linux directly inside Windows, without needing a VM.

Most dbt tutorials, packages, and scripts assume a Linux/Mac environment.

Dependency management (like adapters, Python tools) is cleaner and less error-prone in Linux.

## 1) Create Snowflake trial account
https://signup.snowflake.com/?trial=student



## 2) Install WSL
Install WSL - follow the instruction here
https://learn.microsoft.com/en-us/windows/wsl/install

## 3) Install WSL extensions on VS Code: WSL and Remote Explorer

## 4) Initializa WSL by openning a remote window and connect to WSL

## 5) Install dbt extensio on VS Code

## 6) Install dbt Fusion
curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update    
This downloads and installs the latest dbt-fusion binary into your WSL environment.

Check version:
dbt-fusion --version


## 7) Initialize a new project
mkdir ~/projects && cd ~/projects
dbtf init my_dbt_project
cd my_dbt_project


## 8) Configure Snowflake profile
Edit your dbt profile file in WSL:
nano ~/.dbt/profiles.yml
Example:
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

## 5) Verify connection
dbtf debug
