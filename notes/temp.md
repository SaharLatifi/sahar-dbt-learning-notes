# 📊 Power BI & Git Guide

## Why Can't We Use `.pbix` Files with Git?

The traditional `.pbix` file format is not ideal for version control, and Git integration is not straightforward with `.pbix` files.

A `.pbix` file is a **binary file**. It is essentially a compressed package that can contain the report definition, semantic model, and imported data.

Git is designed to track changes in **text-based files line by line**. When a change is made to a `.pbix` file, Git sees the entire binary file as modified rather than showing exactly what changed.

This makes it difficult to:

* Review individual changes
* Compare different versions
* Understand what was modified
* Resolve merge conflicts
* Collaborate effectively through Git

---

## What Can `.pbip` Do?

The `.pbip` (**Power BI Project**) format makes Power BI projects more suitable for version control.

Instead of storing the report and semantic model in a single binary file, `.pbip` organizes them into separate folders containing text-based definition and metadata files. This allows Git to identify and track specific changes within a Power BI project.

> **Note:** Power BI Project (`.pbip`) is a preview feature and must be enabled in Power BI Desktop before it can be used.

---

## ⚙️ Enable the `.pbip` Feature

Before you can save and work with Power BI reports as `.pbip` projects, the **Power BI Project (`.pbip`)** feature must be enabled in Power BI Desktop.

To enable it:

1. Open **Power BI Desktop**.
2. Go to **File > Options and settings > Options**.
3. Under **Global**, select **Preview features**.
4. Select **Power BI Project (`.pbip`) save option**.
5. Click **OK**.
6. Restart Power BI Desktop.

<img width="743" height="600" alt="image" src="https://github.com/user-attachments/assets/a0942965-5f7a-41cd-a699-063850209bb7" />


After restarting Power BI Desktop, the `.pbip` format will be available when saving a report.

---

## 💾 Save a Power BI Report as a Project

After enabling the `.pbip` feature:

1. Open your Power BI report in **Power BI Desktop**.
2. Go to **File > Save as**.
3. Select **Power BI Project files (`*.pbip`)** as the file type.
4. Choose the project location and save the report.

> 📷 **Screenshot:** Save As → Power BI Project files (`*.pbip`)

Power BI Desktop creates a project folder containing the report and semantic model as separate items.

---

## 📁 Power BI Project Structure

When a report is saved as a Power BI Project, Power BI Desktop separates the report and semantic model into folders.

A typical project structure looks similar to:

```text
MyPowerBIProject/
│
├── MyPowerBIProject.pbip
├── .gitignore
├── MyPowerBIProject.Report/
│   └── ...
└── MyPowerBIProject.SemanticModel/
    └── ...
```

> 📷 **Screenshot:** Power BI project folder structure

### `<project-name>.Report`

This folder contains the **report-specific files**, including:

* Report pages
* Visuals
* Visual properties and positions
* Data fields used by visuals
* Other report definitions

---

### `<project-name>.SemanticModel`

This folder contains the files related to the **semantic model**, including definitions for:

* Tables
* Columns
* Measures
* Relationships
* Roles
* Other model metadata

The semantic model definitions are stored in text-based formats, including **TMDL (Tabular Model Definition Language)**.

---

### `.gitignore`

The `.gitignore` file tells Git which files or folders should **not** be tracked.

Power BI Desktop can create a `.gitignore` file containing entries such as:

```gitignore
**/.pbi/localSettings.json
**/.pbi/cache.abf
```

These files contain local settings or cached information that should not be committed to the repository.

> **Note:** Power BI Desktop creates the `.gitignore` file only when one does not already exist in the selected save folder or its parent Git repository.

If the repository already contains a `.gitignore` file, review it and make sure the required Power BI exclusions are included.

---

### `<project-name>.pbip`

The `.pbip` file is the **main project file** and entry point for the Power BI project.

It points Power BI Desktop to the associated report and semantic model folders.

> **Tip:** To reopen the project in Power BI Desktop, open the `.pbip` file located in the root project folder.

---

## 💻 Open a `.pbip` Project in VS Code

Power BI project definitions are stored as text-based files, primarily using formats such as **JSON** and **TMDL**.

These files can be viewed and, when appropriate, edited directly in **VS Code**.

To open the project in VS Code, open the **root project folder** — the folder containing the `.pbip` file.

> 📷 **Screenshot:** Power BI project opened in VS Code

> **Tip:** Open the entire project folder rather than an individual report or semantic model folder so you can view the complete project structure.

### External Changes

Power BI Desktop may not automatically detect changes made externally while the project is already open.

If project files are modified directly in VS Code, you may need to reopen or restart Power BI Desktop before the changes are reflected.

---

## 🔐 Sensitive Data and Git

The dataset itself is not normally committed to Git as part of the Power BI project. However, a `.pbip` project can still contain **sensitive or confidential information**.

Values written directly into project definitions may be stored as plain text and therefore become visible in Git.

Sensitive information could appear in:

* Power Query (M) code
* Hard-coded filters
* Parameters or connection information
* Content created using **Enter Data**
* Other report or semantic model definitions

### Example

If a client is filtered by PHN directly in Power Query, the PHN may appear as plain text in the M script stored within the Power BI project.

> 📷 **Screenshot:** Example of a hard-coded PHN visible in the Power Query M script in VS Code

If the project is committed, that hard-coded value may also become part of the **Git history**.

> ⚠️ **Important:** Never include PHNs, client identifiers, passwords, access tokens, credentials, connection secrets, or other sensitive information as hard-coded values in Power BI project files.

---

## ✅ Before Committing a Power BI Project

Before committing a Power BI project:

1. Review the changed project files.
2. Check Power Query and other project definitions for hard-coded or sensitive values.
3. Confirm that local and cache files are excluded through `.gitignore`.
4. Make sure no confidential information is included.

> **Tip:** Treat `.pbip` project files like source code. Even though the report data itself may not be stored in Git, text-based definitions can contain values that should not be committed.

For the general Git process, including branching, committing, pushing, pull requests, and merging, follow the team's existing **Git workflow and guidelines**.

