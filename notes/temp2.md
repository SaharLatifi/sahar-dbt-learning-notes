# 📊 Power BI Version Control SOP

## Purpose

This SOP provides the standard workflow for managing Power BI projects with **Git and GitHub Desktop**.

Power BI reports must be saved in the `.pbip` (**Power BI Project**) format so that report and semantic model definitions can be tracked through Git.

---

## 📋 Prerequisites

Before using this workflow, make sure:

- The **Power BI Project (`.pbip`)** feature is enabled in Power BI Desktop.
- **GitHub Desktop** is installed and configured.
- You have access to the appropriate GitHub repository.
- The repository has been cloned to your local computer.

> **Note:** See the **Power BI & Git Guide** for instructions on enabling the `.pbip` feature.

---

# 🆕 First-Time Setup

Use this workflow when adding an existing `.pbix` report to Git for the first time.

## 1. Open the Repository in GitHub Desktop

Open the appropriate repository in **GitHub Desktop**.

If the repository has not already been cloned to your computer, clone it first.

> 📷 **Screenshot:** Repository opened in GitHub Desktop

---

## 2. Sync the `main` Branch

Before starting, make sure your local `main` branch contains the latest version of the repository.

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

> 📷 **Screenshot:** Creating a new branch in GitHub Desktop

---

## 4. Save the Power BI Report as `.pbip`

Open the existing `.pbix` report in **Power BI Desktop**.

1. Go to **File > Save as**.
2. Select **Power BI Project files (`*.pbip`)**.
3. Save the project in the appropriate location **inside the local Git repository**.

Power BI Desktop will create the `.pbip` file along with the associated `.Report` and `.SemanticModel` folders.

> 📷 **Screenshot:** Saving the report as `.pbip`

> ⚠️ **Important:** Make sure the project is saved inside the correct local Git repository so GitHub Desktop can detect and track the project files.

---

## 5. Review the Project Files

Return to **GitHub Desktop** and review the files listed under **Changes**.

Before committing:

1. Confirm that the expected `.pbip`, `.Report`, and `.SemanticModel` files have been added.
2. Confirm that the required Power BI exclusions are included in `.gitignore`.
3. Review the changed files for hard-coded or sensitive information.
4. Make sure only the required project files are included in the commit.

> ⚠️ **Important:** Never commit PHNs, client identifiers, credentials, passwords, connection information, or other sensitive information.

> 📷 **Screenshot:** Power BI project files shown under Changes in GitHub Desktop

---

## 6. Commit the Project

Enter a clear commit message describing the addition of the Power BI project.

Example:

```text
feat: add client summary Power BI project
```

Click **Commit to `<branch-name>`**.

Follow the team's **Commit Guidelines** for commit message conventions.

---

## 7. Publish the Branch

Click **Publish branch** to send the new branch and its commits to GitHub.

> 📷 **Screenshot:** Publish branch in GitHub Desktop

---

## 8. Create a Pull Request

After publishing the branch:

1. Click **Preview Pull Request**.
2. Confirm that the base branch is `main`.
3. Review the changes.
4. Click **Create Pull Request**.
5. Complete the pull request in GitHub.

Follow the team's **Pull Request Guidelines** for pull request requirements.

> 📷 **Screenshot:** Creating a pull request from GitHub Desktop

---

## 9. Merge and Delete the Branch

Once the pull request has been reviewed and approved:

1. Merge the pull request into `main`.
2. Delete the completed branch when it is no longer needed.

Follow the team's **Merge & Delete Guidelines** for the standard process.

The Power BI `.pbip` project is now stored in the repository and managed through Git.

---

# 🔁 Ongoing Power BI Changes

Once a Power BI project has been added to Git, use the following workflow for future changes.

> **Note:** You do not need to save the report as `.pbip` again. Open and work from the existing `.pbip` project stored in your local repository.

## 1. Sync `main`

Before starting a new change:

1. Switch to **main** in GitHub Desktop.
2. Click **Fetch origin**.
3. Click **Pull origin** if updates are available.

This ensures you are starting from the latest approved version of the Power BI project.

---

## 2. Create a New Branch

Create a new branch from the updated `main` branch.

Example:

```text
update-dashboard-filters
```

Follow the team's **Branch Guidelines** for branch naming conventions.

> ⚠️ **Important:** Do not make Power BI changes directly on `main`.

---

## 3. Open the `.pbip` Project

Open the `.pbip` file stored in your local repository.

For example:

```text
ClientSummary/
│
├── ClientSummary.pbip
├── ClientSummary.Report/
└── ClientSummary.SemanticModel/
```

Open:

```text
ClientSummary.pbip
```

This opens the associated report and semantic model in Power BI Desktop.

---

## 4. Make and Save Your Changes

Make the required changes in **Power BI Desktop**.

Save the project when your changes are complete.

---

## 5. Review the Changes

Return to **GitHub Desktop** and review the files listed under **Changes**.

Before committing:

1. Confirm that the changes are expected.
2. Review Power Query and project definitions for hard-coded values.
3. Check for sensitive or confidential information.
4. Make sure only the intended files are included.

> ⚠️ **Important:** Never commit sensitive or confidential information.

> 📷 **Screenshot:** Power BI changes displayed in GitHub Desktop

---

## 6. Commit the Changes

Enter a clear commit message describing the change.

Example:

```text
update: update dashboard filters
```

Click **Commit to `<branch-name>`**.

Follow the team's **Commit Guidelines**.

---

## 7. Push the Changes

Click **Push origin** to send your committed changes to GitHub.

> 📷 **Screenshot:** Push origin in GitHub Desktop

---

## 8. Create a Pull Request

After pushing the changes:

1. Click **Preview Pull Request**.
2. Confirm that the base branch is `main`.
3. Review the changes.
4. Click **Create Pull Request**.
5. Complete the pull request in GitHub.

Follow the team's **Pull Request Guidelines**.

---

## 9. Review and Merge

Once the pull request has been reviewed and approved, merge it into `main`.

The merged version becomes the latest approved version of the Power BI project.

---

## 10. Delete the Completed Branch

Delete the branch once the pull request has been merged and the branch is no longer needed.

Follow the team's **Merge & Delete Guidelines**.

---

## 11. Sync `main`

Return to GitHub Desktop:

1. Switch to **main**.
2. Click **Fetch origin**.
3. Click **Pull origin** if updates are available.

Your local `main` will now contain the latest merged Power BI changes and will be ready for the next change.
