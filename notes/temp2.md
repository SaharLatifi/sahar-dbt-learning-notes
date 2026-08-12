## 🔄 Managing Power BI Version Control with GitHub Desktop

Once the Power BI Project (`.pbip`) feature is enabled, GitHub Desktop can be used to manage the project and maintain its version history.

### First-Time Setup

When adding an existing Power BI report to Git for the first time:

### 1. Open the Repository in GitHub Desktop

Open the appropriate repository in **GitHub Desktop**.

If the repository has not already been cloned to your computer, clone it first.

> 📷 **Screenshot:** Repository opened in GitHub Desktop

---

### 2. Sync the `main` Branch

Before starting, make sure your local repository contains the latest version.

1. Select **Current Branch > main**.
2. Click **Fetch origin**.
3. If updates are available, click **Pull origin**.

> **Tip:** Always update `main` before creating a new branch.

---

### 3. Create a New Branch

Create a branch for the Power BI project or change.

1. Click **Current Branch**.
2. Select **New Branch**.
3. Enter a branch name following the team's branch naming convention.
4. Make sure the branch is based on `main`.
5. Click **Create Branch**.

> 📷 **Screenshot:** Creating a branch in GitHub Desktop

---

### 4. Save the Power BI Report as `.pbip`

Open the existing `.pbix` report in **Power BI Desktop**.

1. Go to **File > Save as**.
2. Select **Power BI Project files (`*.pbip`)**.
3. Save the project in the appropriate location **inside your local Git repository**.

Power BI Desktop will create the `.pbip` file along with the associated `.Report` and `.SemanticModel` folders.

> 📷 **Screenshot:** Saving the report as a `.pbip` project inside the local repository

> **Important:** Make sure the project is saved inside the correct local Git repository so GitHub Desktop can detect and track the project files.

---

## 🔁 Working on Future Power BI Changes

Once the `.pbip` project has been added to the repository, you do **not** need to save it as `.pbip` again.

For future changes:

1. Sync `main`.
2. Create a new branch.
3. Open the existing `.pbip` file in Power BI Desktop.
4. Make the required changes.
5. Save the project.
6. Review the changes in GitHub Desktop.
7. Check for sensitive information.
8. Commit the changes.
9. Push the branch.
10. Create a pull request.
11. Merge and delete the branch.

> **Tip:** The `.pbip` conversion is a **one-time setup**. After that, always open and work from the `.pbip` project stored in the repository.
