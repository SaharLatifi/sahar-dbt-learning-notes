# 📊 Power BI Version Control SOP

## Purpose

This SOP provides the standard workflow for managing Power BI projects with **Git and GitHub Desktop**.

Power BI reports must be saved in the `.pbip` (**Power BI Project**) format so that report and semantic model definitions can be tracked through Git.

---

## 📋 Prerequisites

Before using this workflow, make sure:

* The **Power BI Project (`.pbip`)** feature is enabled in Power BI Desktop.
* **GitHub Desktop** is installed and configured.
* You have access to the appropriate GitHub repository.
* The repository has been cloned to your local computer.

> **Note:** See the **Power BI & Git Guide** for instructions on enabling the `.pbip` feature.

---

# 🔄 Power BI Version Control Workflow

Follow the steps below to manage Power BI projects using **Git and GitHub Desktop**.

## 1. Open the Repository in GitHub Desktop

Open the appropriate repository in **GitHub Desktop**.

If the repository has not already been cloned to your computer, clone it first.

> 📷 **Screenshot:** Repository opened in GitHub Desktop

---

## 2. Sync the `main` Branch

Before starting new work, make sure your local `main` branch contains the latest version of the repository.

1. Select **Current Branch > main**.
2. Click **Fetch origin**.
3. If updates are available, click **Pull origin**.

> **Tip:** Always sync `main` before creating a new branch.

---

## 3. Create a New Branch

Create a new branch from the updated `main` branch.

1. Click **Current Branch**.
2. Select **New Branch**.
3. Enter a branch name following the team's branch naming convention.
4. Make sure the branch is based on `main`.
5. Click **Create Branch**.

Example:

```text
feat-add-client-summary-dashboard
```

Follow the team's **Branch Guidelines** for branch naming conventions.

> ⚠️ **Important:** Do not make Power BI changes directly on `main`.

> 📷 **Screenshot:** Creating a new branch in GitHub Desktop

---

## 4. Open or Add the Power BI Project

At this step, follow the appropriate instructions depending on whether you are adding a Power BI report to Git for the first time or working with an existing `.pbip` project.

### 🆕 Adding a Power BI Report for the First Time

If you are adding an existing `.pbix` report to Git for the first time:

1. Open the `.pbix` report in **Power BI Desktop**.
2. Go to **File > Save as**.
3. Select **Power BI Project files (`*.pbip`)**.
4. Save the project in the appropriate location **inside the local Git repository**.

Power BI Desktop will create the `.pbip` file along with the associated `.Report` and `.SemanticModel` folders.

> ⚠️ **Important:** Make sure the project is saved inside the correct local Git repository so GitHub Desktop can detect and track the project files.

> 📷 **Screenshot:** Saving the report as `.pbip`

### 🔁 Working with an Existing Power BI Project

If the Power BI project is already stored in Git:

1. Open the existing `.pbip` file from your local repository.
2. Make the required changes in **Power BI Desktop**.
3. Save the project when your changes are complete.

> **Note:** You do not need to save the report as `.pbip` again. Continue working from the existing `.pbip` project.

---

## 5. Review the Changes

Return to **GitHub Desktop** and review the files listed under **Changes**.

Before committing:

1. Confirm that the changes are expected.
2. Review Power Query and project definitions for hard-coded values.
3. Check for sensitive or confidential information.
4. Make sure only the intended project files are included.

If you are adding a Power BI project for the first time, also confirm that:

* The expected `.pbip`, `.Report`, and `.SemanticModel` files have been added.
* The required Power BI exclusions are included in `.gitignore`.

> ⚠️ **Important:** Never commit PHNs, client identifiers, credentials, passwords, connection information, or other sensitive information.

> 📷 **Screenshot:** Power BI project changes displayed in GitHub Desktop

---

## 6. Commit the Changes

Enter a clear commit message describing the change.

Example for adding a new Power BI project:

```text
feat: add client summary Power BI project
```

Example for updating an existing project:

```text
update: update dashboard filters
```

Click **Commit to `<branch-name>`**.

Follow the team's **Commit Guidelines** for commit message conventions.

---

## 7. Push the Branch

Send the branch and commits to GitHub.

* For a new branch, click **Publish branch**.
* For a branch that has already been published, click **Push origin**.

> 📷 **Screenshot:** Publish branch / Push origin in GitHub Desktop

---

## 8. Create a Pull Request

After pushing the branch:

1. Click **Preview Pull Request**.
2. Confirm that the base branch is `main`.
3. Review the changes.
4. Click **Create Pull Request**.
5. Complete the pull request in GitHub.

Follow the team's **Pull Request Guidelines**.

> 📷 **Screenshot:** Creating a pull request from GitHub Desktop

---

## 9. Review and Merge

Once the pull request has been reviewed and approved, merge it into `main`.

The merged version becomes the latest approved version of the Power BI project.

---

## 10. Delete the Completed Branch

Once the pull request has been merged and the branch is no longer needed, delete the branch.

Follow the team's **Merge & Delete Guidelines**.

---

## 11. Sync `main`

After the merge:

1. Return to **GitHub Desktop**.
2. Switch to `main`.
3. Click **Fetch origin**.
4. Click **Pull origin** if updates are available.

Your local `main` will now contain the latest approved version of the Power BI project.

---

## ✅ Workflow Summary

```text
Sync main
    ↓
Create branch
    ↓
Open or add Power BI project
    ↓
Make and save changes
    ↓
Review changes
    ↓
Check for sensitive data
    ↓
Commit
    ↓
Push / Publish
    ↓
Create Pull Request
    ↓
Review and merge
    ↓
Delete branch
    ↓
Sync main
```
