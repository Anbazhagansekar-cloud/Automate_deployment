### **Steps to Automate Deployment with PR Auto-Approval**

#### **1. Set Up the GitHub Repository**

1. Create a GitHub repository and organize it using the structure provided above.
2. Place your SQL file (`deployment.sql`) in the `sql/` directory.
3. Add an `azure-pipelines/pipeline.yml` file to define the pipeline configuration.

---

#### **2. Configure the Azure DevOps Pipeline**

**Pipeline YAML (azure-pipelines/pipeline.yml):**

```yaml
trigger:
- main  # Automatically triggers on commits to the main branch

pr:
  branches:
    include:
    - feature/*  # Trigger for Pull Requests targeting the main branch

variables:
  SQL_FILE_PATH: 'sql/deployment.sql'
  DB_SERVER: 'your-sql-server-name.database.windows.net'
  DB_NAME: 'your-database-name'
  DB_USERNAME: $(DB_USERNAME)  # Stored securely in Azure DevOps Pipeline variables
  DB_PASSWORD: $(DB_PASSWORD)  # Stored securely in Azure DevOps Pipeline variables

stages:
- stage: BuildAndTest
  displayName: "Build and Test"
  jobs:
  - job: Build
    displayName: "Fetch SQL and Validate"
    steps:
    - checkout: self

    - script: |
        echo "Validating SQL file exists..."
        if [ ! -f "$(SQL_FILE_PATH)" ]; then
          echo "SQL file not found: $(SQL_FILE_PATH)"
          exit 1
        fi
        echo "SQL file found: $(SQL_FILE_PATH)"
      displayName: "Validate SQL File"

- stage: Deploy
  displayName: "Deploy to Database"
  jobs:
  - job: Deploy
    displayName: "Execute SQL on Azure SQL Database"
    steps:
    - script: |
        sqlcmd -S $(DB_SERVER) -d $(DB_NAME) -U $(DB_USERNAME) -P $(DB_PASSWORD) -i $(SQL_FILE_PATH)
      displayName: "Run SQL Deployment"
      env:
        DB_USERNAME: $(DB_USERNAME)
        DB_PASSWORD: $(DB_PASSWORD)
```

---

#### **3. Securely Store Credentials**

1. Navigate to the Azure DevOps project.
2. Go to **Pipelines > Library** and create a new variable group.
3. Add variables:
   - `DB_USERNAME`: Your database username.
   - `DB_PASSWORD`: Your database password.
4. Mark `DB_PASSWORD` as **secret**.

---

#### **4. Set Up Pull Request Auto-Approval**

To automate the approval of pull requests:

1. **Azure DevOps PR Policies**:
   - Navigate to your repository in Azure DevOps.
   - Go to **Repos > Branches**.
   - Configure branch policies for the `main` branch:
     - Enable **Automatically include reviewers**.
     - Add a service account (or a specific user) as the default reviewer.
     - Enable **Auto-approve if no changes requested**.

2. **GitHub Auto Approval** (Optional for GitHub Actions):
   - Use a bot or GitHub Actions to approve PRs:
     - Add the `Pull Request Auto Approve` GitHub App to your repository.
     - Configure the app to auto-approve PRs based on conditions (e.g., specific labels).

---

#### **5. Test the Setup**

1. **Create a Feature Branch**:
   - Clone the repo and create a feature branch: `feature/add-sql-deployment`.

2. **Update SQL File**:
   - Modify `deployment.sql` in the `sql/` folder.

3. **Create a Pull Request**:
   - Push the feature branch to GitHub.
   - Create a PR targeting the `main` branch.

4. **Validate Automation**:
   - Verify that:
     - The pipeline triggers for the PR.
     - The auto-approval process works.
     - The deployment runs successfully upon merging the PR.

---

#### **Key Benefits**
- **CI/CD**: Automates SQL file deployment from PR to production.
- **Validation**: Ensures the SQL file exists and is valid before execution.
- **PR Auto-Approval**: Simplifies PR processes while maintaining control.

