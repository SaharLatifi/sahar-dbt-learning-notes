# 🟢 dbt Seeds — Key Concepts

## Overview
- A **seed** is a simple way to convert CSV files into database tables. Seeds will be materialized as tables in your data warehouse. 
- Best for **small reference or mapping tables** (typically a few hundred rows).  
- Not intended for:     **large datasets**  and **regularly changable data**
- Good use case: mapping tables like US state abbreviation to state names, country code to names.

---

## About Seed Data
- Seed data lives inside your **dbt project repository** under version control.  
- This means any changes will be tracked and pushed to GitHub.  
- ⚠️ **Never include sensitive data** in seed files, since they are stored in plain text and visible in version history.  

---

## How to Use
1. Create a folder named `/seeds` in your dbt project.  
2. Add your CSV files to this folder.  
   - It’s good practice to name them clearly, such as `seed_mapping.csv` or `seed_country_codes.csv`.  
3. dbt automatically read from seed directory and load these CSV files as table named as the same as file name in your data warehouse
   
***There are configuration option to change the file name, but it is recommended to name the csv file what you would like the table to be called in your warehouse.**   

***dbt will rebuild the table associated with the seeds files each time you invoke dbt seed or build.**


Example structure:
```
/seeds/
 └── seed_country_codes.csv
```

---

## Run Command
Run the following command to load your seed data into the database:

```bash
dbtf seed
dbtf seed --select <<seed name>>
```
✅dbt build command also automatically builds seeds.   

✅dbt builds DAG, and build seeds before any downstream model is also able to use them.

---

## Summary
✅ Converts small CSV files into tables  
✅ Useful for mapping or lookup data  
⚠️ Avoid large or sensitive datasets  
📁 Store seeds in `/seeds` folder, e.g. `seed_xxx.csv`  
🧭 Version-controlled — review changes before pushing to GitHub  
