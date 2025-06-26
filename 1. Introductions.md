# **GitHub Actions - The Complete Guide**

### **1. What is GitHub Actions?**
GitHub Actions is a CI/CD (Continuous Integration/Continuous Deployment) platform that automates workflows triggered by GitHub events (e.g., push, pull request, issue creation).

#### **Key Terms:**
- **Workflow**: A YAML file (`.yml`) defining automation steps.
- **Event**: Triggers a workflow (e.g., `push`, `pull_request`).
- **Job**: A set of steps that run on the same runner (VM).
- **Step**: Individual task (can run a script or action).
- **Action**: Reusable unit of code (e.g., checkout, install dependencies).

---

![1](https://github.com/user-attachments/assets/798632eb-7ad7-4547-8328-497f3c7e95b4)

![2](https://github.com/user-attachments/assets/21d51292-6bad-424d-8096-ac26921cd6ae)

![3](https://github.com/user-attachments/assets/5ad88114-5a18-463b-84c9-517bccb088dc)

![image](https://github.com/user-attachments/assets/90b02078-6929-49db-aebd-9c1f2bc86aa1)

![image](https://github.com/user-attachments/assets/0c8bf205-5f0b-44fa-9b53-3797548489ac)

![image](https://github.com/user-attachments/assets/2b586d7d-359e-4b77-9b39-4b91784a7eb8)

![image](https://github.com/user-attachments/assets/6de56831-b234-4332-b029-3021f2700a0c)

![image](https://github.com/user-attachments/assets/bcf9bf18-27e1-450a-ab2a-7f97746824e0)


---

### **My First Workflow***
```yaml
name: First Workflow
on: [push]

jobs:
  run-shell-command:
    runs-on: ubuntu-latest
    steps:
      - name: Run shell command
        run: echo "Hello, World!"
      
      - name: List files in current directory
        run: ls -la
```
### Run from Github Actions

![image](https://github.com/user-attachments/assets/2abbeec2-b564-4e17-8def-7174cbca2bb9)

![image](https://github.com/user-attachments/assets/3fa9476d-858e-41a8-ab86-7a18bfbc7bca)

---
### **2. Basic Workflow Example**
Create a file at `.github/workflows/main.yml`:
```yaml
name: CI Pipeline
on: [push]  # Trigger on push event

jobs:
  build:
    runs-on: ubuntu-latest  # Runner OS
    steps:
      - name: Checkout code
        uses: actions/checkout@v4  # Official action to fetch code

      - name: Install dependencies
        run: npm install  # Runs a shell command

      - name: Run tests
        run: npm test
```

#### **Explanation:**
- `on: [push]`: Workflow runs on every `git push`.
- `jobs`: Contains one job named `build`.
- `runs-on`: Specifies the VM environment.
- `steps`: Sequential tasks (checkout → install → test).

---

### **3. Key Concepts with Examples**

#### **a) Events**
```yaml
on:
  push:
    branches: [ main ]  # Only on pushes to main
  pull_request:
    branches: [ feature/* ]  # Only PRs to feature branches
```

#### **b) Environment Variables & Secrets**
```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}  # Secret stored in GitHub Settings
    run: echo "Deploying with API key..."
```

#### **c) Matrix Strategy (Parallel Jobs)**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]  # Test across Node.js versions
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
```

#### **d) Caching Dependencies**
```yaml
steps:
  - uses: actions/cache@v3
    with:
      path: node_modules
      key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
```

---

### **4. Real-World Use Cases**
#### **a) Auto-Deploy to GitHub Pages**
```yaml
name: Deploy
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install && npm run build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

#### **b) Notify Slack on Failure**
```yaml
steps:
  - name: Notify Slack
    if: failure()  # Only on job failure
    uses: slackapi/slack-github-action@v1
    with:
      slack-message: "Build failed!"
    env:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

### **5. Important Commands & Tips**
- **Manual Trigger**: Use `workflow_dispatch`:
  ```yaml
  on:
    workflow_dispatch:
      inputs:
        environment:
          description: 'Env to deploy'
          required: true
  ```
- **View Logs**: Go to `Actions` tab in your GitHub repo.
- **Debugging**: Add `run: env` to print all environment variables.

---

### **6. Advanced: Reusable Workflows**
Create a shared workflow (e.g., `.github/workflows/shared.yml`):
```yaml
name: Shared CI
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  build:
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}
```

Call it from another workflow:
```yaml
uses: ./.github/workflows/shared.yml
with:
  node-version: '18.x'
```

---

### **Next Steps**
1. Try automating a test suite for your project.
2. Explore the [GitHub Actions Marketplace](https://github.com/marketplace?type=actions) for pre-built actions.
3. Learn about `needs` for job dependencies.

### **Manual Trigger in GitHub Actions (`workflow_dispatch`)**
To manually trigger a workflow (without relying on `push` or `pull_request`), use the `workflow_dispatch` event. This adds a **"Run workflow"** button in the GitHub UI.

---

### **Example: Basic Manual Trigger**
```yaml
name: Manual Trigger Demo
on:
  workflow_dispatch:  # Allows manual trigger

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Greet
        run: echo "Hello, GitHub Actions! 🎉"
```

#### **How to Run:**
1. Push this workflow to your repo.
2. Go to **Actions** tab → Select the workflow → Click **"Run workflow"**.

---

### **Advanced: Manual Trigger with Inputs**
You can pass inputs (e.g., environment, version) when triggering manually:

```yaml
name: Deploy with Inputs
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      version:
        description: 'App version'
        required: false
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Print inputs
        run: |
          echo "Environment: ${{ github.event.inputs.environment }}"
          echo "Version: ${{ github.event.inputs.version || 'latest' }}"
```

#### **How to Run:**
1. In the GitHub UI, you’ll see a form to fill in inputs:  
   ![Manual Trigger Inputs](https://docs.github.com/assets/images/help/repository/manual-trigger-workflow.png)
2. Click **"Run workflow"** after filling the fields.

---

### **Key Notes:**
1. **`workflow_dispatch` vs `repository_dispatch`**:
   - `workflow_dispatch`: Trigger from GitHub UI or API.
   - `repository_dispatch`: Trigger via API only (useful for external events).

2. **GitHub CLI Alternative**:
   ```bash
   gh workflow run deploy.yml -f environment=production -f version=1.0
   ```

3. **Default Values**: Use `default` for optional inputs.

---

### **Real-World Use Case**
```yaml
name: Deploy to AWS
on:
  workflow_dispatch:
    inputs:
      env:
        description: 'AWS environment'
        options: [dev, prod]
      confirm:
        description: 'Type "yes" to deploy'
        required: true

jobs:
  deploy:
    if: github.event.inputs.confirm == 'yes'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ github.event.inputs.env }}..."
```

This ensures manual confirmation before deployment.

---

# **Parallel and Dependent Jobs in GitHub Actions**

GitHub Actions allows you to run jobs **in parallel** for faster execution or **sequentially** (dependent jobs) where one job waits for another to complete. Below is a detailed breakdown with examples.

---

## **1. Parallel Jobs**
Jobs run in parallel by default (unless dependencies are specified). This is useful for tasks like **testing across multiple OSes or Node.js versions**.

### **Example: Parallel Jobs**
```yaml
name: Parallel Jobs Example
on: [push]

jobs:
  test-linux:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running tests on Linux"

  test-windows:
    runs-on: windows-latest
    steps:
      - run: echo "Running tests on Windows"

  test-macos:
    runs-on: macos-latest
    steps:
      - run: echo "Running tests on macOS"
```
**Key Points:**
- All 3 jobs (`test-linux`, `test-windows`, `test-macos`) run **simultaneously**.
- No dependency means they execute independently.

![image](https://github.com/user-attachments/assets/c35236bd-4473-46ae-bc88-36ed391ae07e)

![image](https://github.com/user-attachments/assets/e11eba46-eb51-4fa3-9ef3-a6e950f33b0e)


![image](https://github.com/user-attachments/assets/fe57df9f-8e08-400e-81cf-7b8af0c5fb01)


---

## **2. Dependent Jobs (Sequential Execution)**
Use `needs` to define job dependencies. A job will only run if its dependent jobs succeed.

### **Example: Sequential Jobs**
```yaml
name: Dependent Jobs Example
on:
  workflow_dispatch:  # Allows manual trigger

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build the application
        run: echo "Building the app..."
      - name: Wait for 30 seconds
        run: sleep 30s  # Linux/macOS (use 'Start-Sleep -Seconds 30' for Windows)

  test:
    runs-on: ubuntu-latest
    needs: build  # Waits for 'build' to finish
    steps:
      - name: Run tests
        run: echo "Running tests..."
      - name: Wait for 30 seconds
        run: sleep 30s  # Linux/macOS (use 'Start-Sleep -Seconds 30' for Windows)


  deploy:
    runs-on: ubuntu-latest
    needs: test   # Waits for 'test' to finish
    steps:
      - name: Deploy to production
        run: echo "Deploying to production..."
      - name: Wait for 30 seconds
        run: sleep 30s  # Linux/macOS (use 'Start-Sleep -Seconds 30' for Windows)

```

![image](https://github.com/user-attachments/assets/64b7f85b-2c9f-4502-8e98-abc9cbe9b38b)

![image](https://github.com/user-attachments/assets/ad8c86dd-d899-46b1-bbba-ea207a6013b9)

![image](https://github.com/user-attachments/assets/87669117-4aec-4f99-ad4c-ef32100957e7)

![image](https://github.com/user-attachments/assets/ab4c04f2-eb1c-409e-a619-b0c12411c4c8)

![image](https://github.com/user-attachments/assets/d21bb024-b9a3-4c1b-aee4-8fd6fee5322e)

**Key Points:**
- `deploy` runs **only after** `test` succeeds.
- `test` runs **only after** `build` succeeds.
- If `build` fails, `test` and `deploy` are **skipped**.

---

## **3. Mixed Parallel & Dependent Jobs**
You can combine both approaches for complex workflows.

### **Example: Parallel Tests → Sequential Deployment**
```yaml
name: Dependent Jobs Example
on:
  workflow_dispatch:  # Allows manual trigger

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build the application
        run: echo "Building the app..."
      - name: Wait for 30 seconds
        run: sleep 30s  # Linux/macOS (use 'Start-Sleep -Seconds 30' for Windows)

  test:
    runs-on: ubuntu-latest
    needs: build  # Waits for 'build' to finish
    steps:
      - name: Run tests
        run: echo "Running tests..."
      - name: Wait for 30 seconds
        run: sleep 30s  # Linux/macOS (use 'Start-Sleep -Seconds 30' for Windows)


  deploy:
    runs-on: ubuntu-latest   # Parallel job
    steps:
      - name: Deploy to production
        run: echo "Deploying to production..."
      - name: Wait for 30 seconds
        run: sleep 30s  # Linux/macOS (use 'Start-Sleep -Seconds 30' for Windows)

```
**Key Points:**
- `build`, `deploy` run **in parallel**.
- `tests` runs **only after build job complete successfully**.

![image](https://github.com/user-attachments/assets/b404dedd-3f57-4198-8d1c-886e13aca2fa)

![image](https://github.com/user-attachments/assets/ed91faaf-62f8-4e87-89c6-f2daeb9a54eb)

---

## **4. Job Status Conditions**
Control job execution based on previous job status using `if`.

### **Example: Conditional Deployment**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "Testing..."

  deploy:
    runs-on: ubuntu-latest
    needs: test
    if: success()  # Only runs if 'test' succeeds
    steps:
      - run: echo "Deploying..."
```

**Key Conditions:**
- `if: success()` → Runs if previous job succeeds.
- `if: failure()` → Runs if previous job fails (useful for notifications).
- `if: always()` → Runs regardless of previous job status.

---

## **5. Matrix Strategy (Parallel Jobs with Variations)**
Run the same job in parallel with different configurations (e.g., OS, Node.js versions).

### **Example: Testing Across Multiple Node.js Versions**
```yaml
name: Matrix Example
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]  # Runs 3 parallel jobs
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```
**Key Points:**
- Creates **3 parallel jobs** for each `node-version`.
- Useful for **cross-version compatibility testing**.

---

## **6. Important Notes**
1. **Job Dependencies**:
   - Use `needs` to define order.
   - Jobs without `needs` run in parallel.
2. **Max Parallel Jobs**:
   - GitHub Free Tier: **20 concurrent jobs**.
   - GitHub Pro/Team: **40 concurrent jobs**.
3. **Job Status**:
   - `success()`, `failure()`, `always()` control execution flow.
4. **Matrix Jobs**:
   - Great for testing multiple environments in parallel.

---

## **Real-World Use Cases**
### **1. CI/CD Pipeline with Parallel Tests**
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint

  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - run: npm test

  deploy:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - run: npm run deploy
```
- `lint` and `test` run in parallel.
- `deploy` waits for both to complete.

### **2. Conditional Deployment on Main Branch**
```yaml
deploy:
  needs: test
  if: github.ref == 'refs/heads/main' && success()
  runs-on: ubuntu-latest
  steps:
    - run: echo "Deploying to production..."
```
- Only deploys if:
  - Branch is `main`.
  - `test` job succeeds.

---

## **Final Summary**
| Concept | Syntax | Use Case |
|---------|--------|----------|
| **Parallel Jobs** | No `needs` | Run independent tasks simultaneously |
| **Dependent Jobs** | `needs: [job1, job2]` | Run jobs in sequence |
| **Matrix Strategy** | `strategy.matrix` | Test across multiple environments |
| **Conditional Jobs** | `if: success()` | Control execution based on status |


---

Debugging workflows in **GitHub Actions** is a crucial step in identifying why a workflow failed or didn't behave as expected. Here's a detailed explanation with examples and important highlights.

---

## 🔍 What is Debugging in GitHub Actions?

Debugging in GitHub Actions means identifying and fixing problems in workflow runs by analyzing logs, using debugging tools, and modifying the workflow for better observability.

---

## 🛠️ Techniques to Debug Workflows

### 1. **Use `ACTIONS_STEP_DEBUG` Secret**

Enable debug logging for **each step's environment and command output**.

#### ✅ How:

* Go to **Settings → Secrets and variables → Actions → New repository secret**
* Name it: `ACTIONS_STEP_DEBUG`
* Value: `true`

#### 📘 Output:

Shows full environment setup and command execution details in logs.

---

### 2. **Use `ACTIONS_RUNNER_DEBUG` Secret**

Enables **runner-level** debug logs. Useful for self-hosted runners or deeper runner internals.

#### ✅ How:

Same process as above, name the secret: `ACTIONS_RUNNER_DEBUG`, set to `true`.

---

### 3. **Use `echo` and `set-output` for Inline Debugging**

#### ✅ Example:

```yaml
- name: Print debugging info
  run: |
    echo "Current directory: $(pwd)"
    echo "List of files:"
    ls -la
```

Use `echo` generously to display key variable values.

---

### 4. **View Raw Logs**

#### ✅ How:

* Open the failed workflow run → Click on a step → Click "View raw logs"
* Helps locate exact output/error location

---

### 5. **Temporary Workflow Steps for Troubleshooting**

Add temporary steps to test parts of the logic.

#### ✅ Example:

```yaml
- name: Check GitHub context
  run: |
    echo "GITHUB_REF: $GITHUB_REF"
    echo "GITHUB_SHA: $GITHUB_SHA"
```

---

### 6. **Use `continue-on-error: true`**

This allows a step to fail but continue the workflow—great for debugging without full stop.

#### ✅ Example:

```yaml
- name: Run test suite
  run: npm test
  continue-on-error: true
```

---

### 7. **Use Matrix Debugging**

Test different OS/node versions/etc. in parallel.

#### ✅ Example:

```yaml
strategy:
  matrix:
    node: [14, 16, 18]
```

If only one fails, the issue is isolated.

---

## 🧪 Full Debugging Example

```yaml
name: Debug Workflow

on: [push]

jobs:
  debug-example:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Debug Environment
        run: |
          echo "Branch: $GITHUB_REF"
          echo "Commit: $GITHUB_SHA"
          echo "Files in repo:"
          ls -la

      - name: Test Failing Step
        run: |
          echo "Intentional fail"
          exit 1
        continue-on-error: true

      - name: Final Step
        run: echo "Continuing after failure"
```

---

## ✅ Best Practices for Debugging

| Tip                         | Description                                              |
| --------------------------- | -------------------------------------------------------- |
| 🧵 Add `ACTIONS_STEP_DEBUG` | Enables verbose logs for every step                      |
| 🧹 Clean up after debugging | Remove debug-only steps and secrets                      |
| 🔁 Rerun failed workflows   | Use "Re-run failed jobs" for faster feedback             |
| 📦 Use cache carefully      | Corrupt cache may cause strange bugs – try clearing it   |
| 🧪 Use `if: always()`       | Ensure post-step runs even on failure for log collection |

---

Great! Setting `ACTIONS_STEP_DEBUG=true` helps enable **step-level debugging** in GitHub Actions.

---

## 🔍 What Does `ACTIONS_STEP_DEBUG=true` Do?

When you create a secret named `ACTIONS_STEP_DEBUG` with the value `true`, GitHub enables **verbose logs for each step**, including:

* Full command output (even those usually hidden)
* Environment variable evaluations
* Internal execution metadata

It helps you understand **what exactly is being run** and **why something might be failing**.

---

## 🧪 Example: How to Use It

### ✅ Step 1: Add the secret

1. Go to your repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **"New repository secret"**
3. Name: `ACTIONS_STEP_DEBUG`
   Value: `true`

---

### ✅ Step 2: Trigger the Workflow

Push a commit or manually trigger a workflow using `workflow_dispatch`:

```yaml
on:
  workflow_dispatch:
```

---

### ✅ Step 3: Check Logs

After running the workflow:

* Go to **Actions → Workflow Run → Job → Step**
* You’ll see expanded output, including:

  * Environment variables
  * Actual shell commands run
  * Any errors in more detail

---

## 📘 Example Output Before and After

### ❌ *Without* `ACTIONS_STEP_DEBUG`:

```bash
Run npm install
> Installing dependencies...
```

### ✅ *With* `ACTIONS_STEP_DEBUG=true`:

```bash
##[debug]Evaluating condition for step: 'Install dependencies'
##[debug]Set workingDirectory = '/home/runner/work/my-repo'
> Run npm install
> Executing command: npm install
> STDOUT: ...
> STDERR: ...
```

---

## 🔐 Security Note

`ACTIONS_STEP_DEBUG` is a **secret**, so:

* It's **not printed** in logs
* It can be **used selectively**, removed after debugging

---

Great — let's walk through a **real workflow example** where we enable `ACTIONS_STEP_DEBUG=true` to debug a failing step.

---

## ✅ Scenario: NPM install is failing in a Node.js project

### 🔧 Workflow File: `.github/workflows/node-debug.yml`

```yaml
name: Node.js CI Debug

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Show environment debug info
        run: |
          echo "GITHUB_REF: $GITHUB_REF"
          echo "NODE_VERSION: $(node -v)"
          echo "NPM_VERSION: $(npm -v)"
          echo "Current Dir:"
          pwd
          echo "Files:"
          ls -la

      - name: Install dependencies
        run: |
          echo "Installing NPM packages"
          npm install

      - name: Run tests
        run: |
          echo "Running tests"
          npm test
```

---

## 🔐 Secret Setup

Create a repository secret:

| Name                 | Value  |
| -------------------- | ------ |
| `ACTIONS_STEP_DEBUG` | `true` |

> This will **not require code change**—GitHub automatically picks it up during workflow execution.

---

## 🧪 Example Debug Log Output

After running the workflow, you'll see enhanced logs:

```bash
##[debug]Evaluating condition for step: 'Install dependencies'
##[debug]Current working directory: /home/runner/work/node-app
> Installing NPM packages
> npm install
##[debug]Exit code 1 received from tool 'npm'
npm ERR! Failed to fetch module...
```

You’ll now be able to:

* See what `npm install` actually ran
* View actual paths, versions, env variables
* Debug why modules failed to install (network, auth, version mismatch, etc.)

---

## ✅ Final Tips

| Tip                                 | Why It Helps                                          |
| ----------------------------------- | ----------------------------------------------------- |
| Use `echo` for critical variables   | Check values like `$GITHUB_REF`, paths, etc.          |
| Add `continue-on-error: true`       | Let workflows continue past failure for full context  |
| Combine with `ACTIONS_RUNNER_DEBUG` | For runner-level issues (esp. on self-hosted runners) |
| Remove secrets after debugging      | To avoid verbose logs in production                   |

---


Skipping workflows in **GitHub Actions** means conditionally preventing a workflow or a job or a step from running — based on specific criteria like file changes, commit messages, branch names, or manual inputs.

---

## 🟨 Why Skip Workflows?

To:

* Save CI/CD resources
* Avoid unnecessary runs on doc or README updates
* Prevent redundant runs on specific branches
* Allow manual control over executions

---

## ✅ Ways to Skip Workflows

---

### 🔹 1. **Skip via Commit Message (`[skip ci]`)**

#### 🔧 Behavior:

If your commit message contains `[skip ci]` or `[ci skip]`, **GitHub Actions will not run any workflow at all**.

#### ✅ Example:

```bash
git commit -m "Update README [skip ci]"
```

🟡 **Works globally** across all workflows.

---

### 🔹 2. **Use `if:` Conditions at Workflow Level**

You can stop a job or step using `if:` conditions.

#### ✅ Example: Skip workflow if not on `main` or `develop`

```yaml
jobs:
  build:
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Only runs on main or develop"
```

📌 **Important**: `github.ref` contains the full `refs/heads/<branch>`.

---

### 🔹 3. **Use `paths` / `paths-ignore` to Skip Based on File Changes**

In the `on:` trigger section, use these filters:

#### ✅ Example: Run only when `src/` changes

```yaml
on:
  push:
    paths:
      - 'src/**'
```

#### ✅ Example: Ignore changes to docs

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
      - '**.md'
```

📘 **Use Case**: Skip when only non-code files are changed.

---

### 🔹 4. **Use Manual Input to Skip with `workflow_dispatch`**

#### ✅ Example: Use an input to skip optional jobs

```yaml
on:
  workflow_dispatch:
    inputs:
      run_tests:
        type: boolean
        default: true
        description: 'Run test suite?'

jobs:
  test:
    if: ${{ inputs.run_tests == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running tests"
```

🧪 **Try**: Trigger the workflow manually and uncheck the `run_tests` input.

---

### 🔹 5. **Use `continue-on-error` + Early Exit**

Sometimes you want to **exit a job or step early** without failing.

#### ✅ Example:

```yaml
- name: Conditional exit
  run: |
    if [ "$GITHUB_REF" != "refs/heads/main" ]; then
      echo "Skipping on non-main branch"
      exit 0
    fi
    echo "Continue logic here"
  continue-on-error: true
```
---
Here's a **short note** on **skipping workflow runs in GitHub Actions**:

---

## 🔹 Skipping Workflow Runs – Summary

### ✅ **Supported Events**

Skip instructions apply only to workflows triggered by:

* `push`
* `pull_request`

---

### ✅ **How to Skip**

Add any of the following **to the commit message**:

```
[skip ci]
[ci skip]
[no ci]
[skip actions]
[actions skip]
```

OR

Use a **trailer** at the end of the commit message:

```
<commit message>


skip-checks: true
```

> Use `--cleanup=verbatim` with `git commit` to preserve blank lines.

---

### ⚠️ Important Notes

* ❌ Does **not** work for `pull_request_target` or other events.
* ✅ Skipped workflows show **"Pending"** checks – blocking PR merges if required checks are enforced.
* ✅ To unblock, push another commit **without** skip keywords.

---

Let me know if you'd like a real example commit or workflow showing this.

---

## ✅ Summary Table

| Technique                | Scope          | When to Use                                |
| ------------------------ | -------------- | ------------------------------------------ |
| `[skip ci]`              | Whole workflow | Skip entirely via commit message           |
| `if:` condition          | Job/Step       | Skip based on branch, input, env, etc.     |
| `paths` / `paths-ignore` | Trigger-level  | Skip based on changed files                |
| `workflow_dispatch`      | Trigger-level  | Use manual inputs to control job execution |
| Early exit in `run` step | Step-level     | Skip via script logic inside the workflow  |

---

## 💡 Best Practices

* ✅ Combine `paths-ignore` with `branches` for fine-grained control.
* ✅ Use `workflow_dispatch` inputs for manual override workflows.
* ✅ Don’t rely **only** on `[skip ci]` — it's not flexible or explicit in code.
* ✅ Use `if:` for logic that's easier to maintain and debug.

---


---

Here’s a complete and clear explanation of **workflow commands** in GitHub Actions, with good examples and key highlights.

---

## 🔧 What Are Workflow Commands?

**Workflow commands** are special `echo` statements in GitHub Actions that allow a step to:

* Set environment variables
* Mask secrets
* Add logging annotations
* Control step output

They are used **within `run` steps** to communicate with the GitHub Actions runner.

---

## 📌 Syntax

```bash
echo "::command-name parameter=value::message"
```

---

## 📘 Common Workflow Commands with Examples

---

### 1. 🧪 `set-output` (Deprecated → Use `outputs` + `GITHUB_OUTPUT` instead)

#### ✅ Old:

```bash
echo "::set-output name=my_var::Hello"
```

#### ✅ New Way (Preferred):

```yaml
- name: Set output
  id: example
  run: echo "my_var=Hello" >> "$GITHUB_OUTPUT"

- name: Use output
  run: echo "Output was ${{ steps.example.outputs.my_var }}"
```

---

### 2. 🔒 `add-mask` — Mask sensitive data in logs

#### ✅ Example:

```yaml
- name: Mask a secret
  run: |
    echo "::add-mask::my-super-secret-token"
    echo "This will be masked: my-super-secret-token"
```

🔐 Output:

```
This will be masked: ***
```

---

### 3. 🧪 `set-env` (Deprecated → Use `GITHUB_ENV` instead)

#### ✅ Preferred:

```yaml
- name: Set env variable
  run: echo "MY_ENV=foobar" >> "$GITHUB_ENV"

- name: Use env
  run: echo "Env is $MY_ENV"
```

---

### 4. 📘 `warning`, `error`, `debug` — Annotate logs

#### ✅ Example:

```yaml
- name: Add log annotations
  run: |
    echo "::warning file=app.js,line=10::Deprecated function"
    echo "::error file=main.py,line=5::Syntax error"
    echo "::debug::Only visible with ACTIONS_STEP_DEBUG=true"
```

```yaml
name: Workflow Commands
on: [push]

jobs:
  testing-wf-commands:
    runs-on: ubuntu-latest
    steps:
      - name: Setting an error message
        run: echo "::error::Missing semicolon"
      - name: Setting an error message with params
        run: echo "::error title=Error title,file=app.js,line=2,endLine=3,col=5,endColumn=7::Missing Semicolon"
      - name: Setting a debug message with params
        run: echo "::debug title=Debug title,file=app.js,line=2,endLine=3,col=5,endColumn=7::Missing Semicolon"
      - name: Setting an warning message with params
        run: echo "::warning title=Warning title,file=app.js,line=2,endLine=3,col=5,endColumn=7::Missing Semicolon"
      - name: Setting an notice message with params
        run: echo "::notice title=Notice title,file=app.js,line=2,endLine=3,col=5,endColumn=7::Missing Semicolon"
      - name: Group of logs
        run: |
          echo "::group::My group title"
          echo "Inside group"
          echo "::endgroup::"
      - name: Masking a value
        run: echo "::add-mask::Secret String"
      - name: Echo a secret
        run: echo "Secret String"
```

🔎 Output:

* 🟡 Warning with file & line reference
* 🔴 Error
* 🐞 Debug (only with `ACTIONS_STEP_DEBUG=true`)

---

### 5. ✅ `stop-commands` — Temporarily disable interpretation

#### ✅ Example:

```yaml
- name: Stop interpreting workflow commands
  run: |
    echo "::stop-commands::stopmarker"
    echo "::set-env name=SHOULD_NOT_BE_SET::bad"
    echo "This will not be interpreted"
    echo "::stopmarker::"
```

📌 Used when output may accidentally contain command syntax.

---

## 💡 Key Points to Remember

| Feature                | Use it for                          | Status      |
| ---------------------- | ----------------------------------- | ----------- |
| `GITHUB_OUTPUT`        | Passing values between steps        | ✅ Preferred |
| `add-mask`             | Hiding sensitive info in logs       | ✅ Important |
| `GITHUB_ENV`           | Set env variables                   | ✅ Preferred |
| `::warning`, `::error` | Annotating logs with context        | ✅ Useful    |
| `stop-commands`        | Prevent misinterpretation in output | ✅ Rare      |
| `::debug::`            | Log only in debug mode              | ✅ Optional  |

---

## 🔧 Use Case Example: Pass Value Between Steps

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Set output value
        id: set_step
        run: echo "greeting=Hello, World!" >> "$GITHUB_OUTPUT"

      - name: Print output
        run: echo "Message: ${{ steps.set_step.outputs.greeting }}"
```

---

Here are **a few complete and practical examples** of using **GitHub Actions workflow commands** inside a full `workflow.yml`. These show real-world usage like setting environment variables, passing data between steps, masking secrets, and adding log annotations.

---

## ✅ **Example 1: Set Output from One Step and Use in Another**

```yaml
name: Set and Use Output

on: [workflow_dispatch]

jobs:
  output-example:
    runs-on: ubuntu-latest

    steps:
      - name: Set output value
        id: set_version
        run: echo "version=1.2.3" >> "$GITHUB_OUTPUT"

      - name: Use output
        run: echo "App version is ${{ steps.set_version.outputs.version }}"
```
![image](https://github.com/user-attachments/assets/d32b6a76-4946-49cb-9be2-d82c70364429)

![image](https://github.com/user-attachments/assets/cf3abcb9-c3ea-44b1-b581-fcc3c32a22ae)

---

🔁 **Use case**: Pass a version or build number between steps.

---

## ✅ **Example 2: Set Environment Variable Across Steps**

```yaml
name: Set Env Variable

on: [workflow_dispatch]

jobs:
  env-example:
    runs-on: ubuntu-latest

    steps:
      - name: Define environment variable
        run: echo "DEPLOY_ENV=production" >> "$GITHUB_ENV"

      - name: Use environment variable
        run: echo "Deploying to $DEPLOY_ENV"
```
![image](https://github.com/user-attachments/assets/4e303533-06b5-440e-ac1d-2170efb46f9e)


---

### 👇 Line-by-Line Explanation

#### ✅ Step 1: Define environment variable

```yaml
- name: Define environment variable
  run: echo "DEPLOY_ENV=production" >> "$GITHUB_ENV"
```

* **Purpose**: This sets a custom environment variable (`DEPLOY_ENV`) for later steps in the same job.
* **How**: It appends `DEPLOY_ENV=production` to the special file `$GITHUB_ENV`.
* **What is `$GITHUB_ENV`?**
  It's a GitHub Actions-specific environment file. Any key-value pair written into it becomes available to **all subsequent steps** as an environment variable.

---

#### ✅ Step 2: Use the environment variable

```yaml
- name: Use environment variable
  run: echo "Deploying to $DEPLOY_ENV"
```

* **Purpose**: This uses the environment variable that was defined in the previous step.
* **Output**: It will print:

  ```
  Deploying to production
  ```

---

### ✅ Summary

* `>> "$GITHUB_ENV"` → Safely persists env vars across steps.
* `$DEPLOY_ENV` → Accessible in all **later** steps once defined.
* Order **matters**: The variable must be defined before it's used.
  

---

If you **don’t use** `>> "$GITHUB_ENV"` to define the variable, here’s what happens:

---

### ❌ Without `$GITHUB_ENV`

```yaml
- name: Define environment variable
  run: |
    export DEPLOY_ENV=production

- name: Use environment variable
  run: echo "Deploying to $DEPLOY_ENV"
```

### 🚫 Problem

* This **won’t work** as expected.
* **Why?**
  Each `run` step in GitHub Actions executes in its **own shell session**.
  So `export DEPLOY_ENV=production` only applies **within that step** — it **does not persist** to the next step.

---

### ✅ Why `>> "$GITHUB_ENV"` is needed

```bash
echo "DEPLOY_ENV=production" >> "$GITHUB_ENV"
```

* This **writes the variable** to a file that GitHub Actions **reads automatically between steps**.
* It’s the **official way** to persist environment variables **across steps** in the same job.

---

### 🔄 Correct vs Incorrect

| Step Definition                   | Persisted Across Steps? | Works in Next Step? |
| --------------------------------- | ----------------------- | ------------------- |
| `export VAR=value` in `run`       | ❌ No                    | ❌ No                |
| `echo "VAR=value" >> $GITHUB_ENV` | ✅ Yes                   | ✅ Yes               |

---

Let me know if you'd like to compare this with job-level or workflow-level environment variables too!



---
🌍 **Use case**: Define and use variables like environment (`dev`, `staging`, `prod`) across multiple steps.

---

## ✅ **Example 3: Mask Sensitive Values in Logs**

```yaml
name: Mask Secrets

on: [workflow_dispatch]

jobs:
  mask-example:
    runs-on: ubuntu-latest

    steps:
      - name: Mask a token
        run: |
          echo "::add-mask::my-secret-token-123"
          echo "Using token: my-secret-token-123"
```

🔐 **Use case**: Hide sensitive tokens from showing up in workflow logs.

![image](https://github.com/user-attachments/assets/5e5e451a-787f-4bd9-ba41-128e2433e324)

---

## ✅ **Example 4: Add Warning and Error Annotations**

```yaml
name: Log Annotations

on: [workflow_dispatch]

jobs:
  log-annotations:
    runs-on: ubuntu-latest

    steps:
      - name: Annotate logs
        run: |
          echo "::warning file=main.js,line=42::Deprecated API usage"
          echo "::error file=main.js,line=99::Null pointer exception"
          echo "::debug::This is a debug message (shown only with ACTIONS_STEP_DEBUG=true)"
```

📘 **Use case**: Highlight issues in logs for better visibility during CI runs.

![image](https://github.com/user-attachments/assets/a6b59348-5aed-4b83-8ee5-c605cba4b09e)

---

## ✅ **Example 5: Skip Interpreting Workflow Commands**

```yaml
name: Stop Commands

on: [workflow_dispatch]

jobs:
  stop-commands-example:
    runs-on: ubuntu-latest

    steps:
      - name: Stop command interpretation
        run: |
          echo "::stop-commands::my-end"
          echo "::set-env name=SHOULD_NOT_RUN::hello"
          echo "This won't be interpreted"
          echo "::my-end::"
          echo "Now commands work again"
```

🛑 **Use case**: Prevent accidental interpretation of dangerous-looking command output.

---
Perfect! Here's a **real-world multi-step GitHub Actions workflow** combining:

* ✅ `Build/Test workflow`
* ✅ `Deployments`
* ✅ `Matrix jobs`
* ✅ `Conditional logic`
* ✅ Workflow commands (`GITHUB_ENV`, `GITHUB_OUTPUT`, `add-mask`, logging)

---

## 🔧 **Full Real-World Workflow Example**

```yaml
name: CI/CD Pipeline

on:
  workflow_dispatch:

jobs:
  build-and-test:
    name: Build & Test (Node.js Matrix)
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [16, 18]
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: |
          npm test
          echo "test_result=success" >> $GITHUB_OUTPUT
        id: test

      - name: Set environment stage
        run: echo "STAGE_ENV=staging" >> $GITHUB_ENV

  deploy:
    name: Deploy to Production
    needs: build-and-test
    runs-on: ubuntu-latest
    if: ${{ always() && contains(needs.build-and-test.result, 'success') }}
    steps:
      - name: Mask deploy token
        run: echo "::add-mask::my-deploy-token-abc123"

      - name: Set deploy version
        run: echo "deploy_version=1.2.${{ github.run_number }}" >> $GITHUB_OUTPUT
        id: version

      - name: Echo logs with annotations
        run: |
          echo "::warning::Deployment will proceed to PROD"
          echo "::debug::STAGE_ENV is $STAGE_ENV"
          echo "Deploying version ${{ steps.version.outputs.deploy_version }} using token ***"
```
![image](https://github.com/user-attachments/assets/3e958c3b-6b29-42cb-837a-ce9bfe1d0d31)

---

## 🔍 What's Happening Here?

| Feature                  | Where Used                                             |
| ------------------------ | ------------------------------------------------------ |
| 🔄 **Matrix job**        | `node: [16, 18]` — runs build/test in parallel         |
| 📦 **Build/Test**        | `npm install` + `npm test`                             |
| 🔐 **Masking secrets**   | `add-mask::my-deploy-token-abc123`                     |
| 🌍 **Set env variable**  | `echo STAGE_ENV=staging >> $GITHUB_ENV`                |
| 📤 **Set output**        | `echo deploy_version=... >> $GITHUB_OUTPUT`            |
| ⚠️ **Log annotation**    | `::warning::`, `::debug::`                             |
| ✅ **Conditional deploy** | `if: contains(needs.build-and-test.result, 'success')` |

---

### Link
[Userguide for Workflow Commands](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions)

## 🔐 Tip for Secrets in Real CI/CD

Replace hardcoded tokens like `my-deploy-token-abc123` with GitHub secrets:

```yaml
- name: Mask token
  run: echo "::add-mask::${{ secrets.DEPLOY_TOKEN }}"
```

---

Great start! Let’s break down **“Shell and Working Directories in GitHub Actions”** in your notes style.

---

### 🔧 Topic: **Shell and Working Directories in GitHub Actions**

---

### 1️⃣ **What is it?**

GitHub Actions uses **shells** to run commands inside the `run:` step. The **default shell** depends on the OS runner:

| Runner OS        | Default Shell |
| ---------------- | ------------- |
| `ubuntu-latest`  | `bash`        |
| `windows-latest` | `powershell`  |
| `macos-latest`   | `bash`        |

You can override this to use `bash`, `sh`, `pwsh`, `cmd`, or even custom shells.

A **working directory** is the folder from which your shell commands run. By default, it’s the root of your repository, but you can change it per step or job.

---

### 2️⃣ **Why is it needed?**

* To **run scripts** using a specific shell (like `bash` or `pwsh`).
* To **organize commands** to run from a specific folder (e.g., inside `/backend`, `/frontend`).
* Helps prevent file not found errors due to incorrect paths.

---

### 3️⃣ **Real-Life Scenario**

> You're building a monorepo with separate folders: `/api`, `/frontend`, `/infrastructure`. You only want to run build scripts inside `api` using `bash`.

---

### 4️⃣ **YAML Example – Using shell & working-directory**

```yaml
name: Shell and Working Directory Example

on: [push]

jobs:
  run-scripts:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Bash command from root
        run: echo "Hello from root using bash"
        shell: bash

      - name: Run command inside /api folder
        run: npm install && npm run build
        working-directory: ./api
        shell: bash

      - name: Run PowerShell on Windows
        if: runner.os == 'Windows'
        run: Write-Host "This is PowerShell"
        shell: pwsh
```

---

### 5️⃣ **Pro Tips**

* You can **globally override** the shell at the job level (but not working-directory):

  ```yaml
  jobs:
    job1:
      runs-on: ubuntu-latest
      defaults:
        run:
          shell: bash
  ```

* For multi-folder repos, use `working-directory:` to avoid `cd` commands.

* Avoid Windows-only shells (like `cmd`) unless you **target `windows-latest`** runner.

---

Perfect, Deepak! Here's how we'll proceed:

---

### ✅ Working Style Agreed

* For each topic, I’ll give:

  * **📘 Concept Explanation**
  * **💡 Real-life Use Cases**
  * **📂 Multiple YAML Examples** (basic → intermediate)
  * **⚠️ Common Mistakes**
  * **🛠 Pro Tips**

---

### 🗂 Topics Log (We'll build this together)

| ✅  | Topic                                             | Status      |
| -- | ------------------------------------------------- | ----------- |
| ✔️ | Shell & Working Directory                         | In Progress |
| ⏳  | `env` and `secrets`                               | Pending     |
| ⏳  | `jobs` and `steps` structure                      | Pending     |
| ⏳  | `matrix` builds                                   | Pending     |
| ⏳  | `if` conditions & expressions                     | Pending     |
| ⏳  | Artifacts                                         | Pending     |
| ⏳  | Caching                                           | Pending     |
| ⏳  | Reusable & Composite Actions                      | Pending     |
| ⏳  | GitHub Contexts (`github`, `env`, `runner`, etc.) | Pending     |
| ⏳  | Deployments (e.g., to S3, EC2, Azure, etc.)       | Pending     |

You can add to this list anytime.

---

Now let’s give **Shell and Working Directories** a **more in-depth** treatment as promised:

---

## 🔧 Topic: Shell and Working Directories in GitHub Actions

---

### 📘 1. What is a Shell?

A **shell** is the command-line interpreter that GitHub Actions uses to run your commands. Each operating system runner has a default shell:

| OS      | Default Shell            |
| ------- | ------------------------ |
| Ubuntu  | `bash`                   |
| Windows | `pwsh` (PowerShell Core) |
| macOS   | `bash`                   |

You can override it with:

* `bash`
* `sh`
* `pwsh` (PowerShell Core)
* `cmd` (Windows Command Prompt)
* `python` (advanced, but possible for scripts)

---

### 📘 2. What is `working-directory`?

* The `working-directory` key specifies the folder to **run the command from**.
* By default, it is the **root of your GitHub repository** (`$GITHUB_WORKSPACE`).
* Helps when you're working in **monorepos**, microservices, or specific folders.

---

### 💡 3. Real-World Use Cases

1. **Monorepo Build Pipelines**

   * Run `npm install` only in `frontend/` or `api/`.

2. **Infrastructure as Code**

   * Run `terraform plan` in `infra/aws/`.

3. **Cross-platform Shell Logic**

   * Run `.bat` script in Windows, `.sh` in Ubuntu.

4. **CI/CD Pipelines with Mixed Tech**

   * Python in `/data`, Java in `/backend`, Bash in root.

---

### 🧪 4. Basic to Advanced Examples

#### 🟢 **Basic Example – Bash in Ubuntu**

```yaml
name: Bash Hello World

on: [push]

jobs:
  bash-example:
    runs-on: ubuntu-latest
    steps:
      - name: Print with Bash
        run: echo "This is Bash"
        shell: bash
```

---

#### 🟡 **Intermediate – Run from Subdirectory**

```yaml
name: Subdirectory Build

on: [push]

jobs:
  node-build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install & Build from /frontend
        run: |
          npm install
          npm run build
        working-directory: frontend
```

---

#### 🔵 **Cross-platform Shells**

```yaml
name: OS-specific Shells

on: [push]

jobs:
  windows:
    runs-on: windows-latest
    steps:
      - name: Run PowerShell
        run: Write-Host "Windows Hello"
        shell: pwsh

  linux:
    runs-on: ubuntu-latest
    steps:
      - name: Run Shell
        run: echo "Linux Hello"
        shell: bash
```

---

#### 🔴 **Global Shell for All Steps**

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
    steps:
      - run: echo "This and all steps use bash"
```

---

### ⚠️ 5. Common Pitfalls

* ❌ Running PowerShell on Ubuntu runner:

  ```yaml
  shell: pwsh  # only works if pwsh is installed on that runner
  ```
* ❌ `working-directory` path doesn't exist → job fails.
* ❌ Forgetting to use `./` or full path in `working-directory`.

---

### 🛠 6. Pro Tips

* Use `defaults.run.shell` and `defaults.run.working-directory` to reduce duplication:

  ```yaml
  defaults:
    run:
      shell: bash
      working-directory: backend
  ```

* Use `pwd` or `echo $PWD` to debug your working directory.

* Use matrix builds to run same steps in different directories or with different shells.

```yaml
name: Working Dirs & Shells
on: [push]
defaults:
  run:
    shell: bash
    # working-directory: /de/ed
jobs:
  display-wd-info:
    runs-on: ubuntu-latest
    steps:
      - name: Display Working Directory & List Files
        run: |
          pwd
          ls -a
          echo $GITHUB_SHA
          echo $GITHUB_REPOSITORY
          echo $GITHUB_WORKSPACE
      - name: Change Working Dir
        working-directory: /home/runner
        run: pwd
  display-wd-info-windows:
    runs-on: windows-latest
    defaults:
      run:
        shell: pwsh
    steps:
      - name: Display Working Directory & List Files
        run: |
          Get-Location
          dir
          echo $env:GITHUB_SHA
          echo $env:GITHUB_REPOSITORY
          echo $env:GITHUB_WORKSPACE
      - name: Python Shell
        shell: python
        run: |
          import platform 
          print(platform.processor())
```
![image](https://github.com/user-attachments/assets/faabd3d2-07fd-4ec9-88f2-6a42766d6f51)

![image](https://github.com/user-attachments/assets/1c402124-a62c-4e84-a87a-78ba518dbaf2)


---

✅ That’s your **detailed notebook entry** on **Shell & Working Directory** in GitHub Actions.

---

Great follow-up question, Deepak! Downloading your GitHub repo onto the **runner machine** is a foundational step for almost every GitHub Actions workflow.

---

## 🔧 Topic: **How to Download a GitHub Repository into the Runner**

---

### 📘 1. What’s Happening Here?

When a GitHub Actions workflow runs, it spins up a **runner machine** (a fresh virtual machine). But this VM doesn't **automatically** have your repo's code unless you explicitly **check it out**.

👉 You use the **`actions/checkout`** action to **download (clone)** your repo into the runner workspace.

---

### 💡 2. Real-Life Use Case

You're setting up a CI/CD pipeline for:

* Running unit tests
* Linting code
* Building Docker images
* Deploying infrastructure (e.g., `terraform`)

👉 All these steps need the actual source code → Hence, you first **clone the repo using `actions/checkout@v4`**.

---

### 📦 3. Basic Example

```yaml
name: Checkout Example

on: [workflow_dispatch]

jobs:
  checkout-job:
    runs-on: ubuntu-latest

    steps:
      - name: List Files Before
        run: ls -lahrt
      
      - name: ✅ Checkout Code
        uses: actions/checkout@v4

      - name: 📂 List Files in Repo After
        run: |
          ls -lahrt
          pwd

```

⏱ After `checkout`, your repo is downloaded to the **default working directory**:
`/home/runner/work/<repo-name>/<repo-name>`

![image](https://github.com/user-attachments/assets/2e034a89-f5ac-474b-a23b-7d01ce67d7f4)

![image](https://github.com/user-attachments/assets/1544d211-68e5-48de-a84c-ff234f7dce88)

![image](https://github.com/user-attachments/assets/71d4cd0a-8cbd-49aa-aadd-6dd2cf1a75d0)


---

### 🔍 4. What `actions/checkout@v4` Does

* Clones your repository at the commit that triggered the workflow.
* Sets up Git credentials so other steps can use Git safely.
* Caches some data to speed up repeated checkouts.

---

### ⚙️ 5. Advanced Usage (Options)

#### 🟡 Checkout a Specific Branch

```yaml
- uses: actions/checkout@v4
  with:
    ref: feature/login-api
```

#### 🟢 Checkout with Submodules

```yaml
- uses: actions/checkout@v4
  with:
    submodules: true
```

#### 🔴 Checkout a Different Repository

```yaml
- uses: actions/checkout@v4
  with:
    repository: deepak-devops/infra-scripts
    token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
    path: infra
```

> Useful in mono-repo setups or when pulling infra/deployment code from another repo.

---

### 🧠 6. Behind the Scenes

The checkout action:

* Uses a shallow clone (`--depth=1`) by default (you can disable this if needed).
* Uses `GITHUB_TOKEN` to access private repos securely.

---

### ⚠️ 7. Common Pitfalls

* ❌ **Forgetting to add `actions/checkout`** → later steps will fail (`file not found`, `npm ERR! missing package.json`, etc.)
* ❌ **Wrong `ref` value** → ends up on wrong branch or commit.
* ❌ **Forgetting `token:` when checking out private repos**

---

### 🛠 8. Pro Tips

* If your workflows **don’t need the full git history**, keep `fetch-depth: 1` (default) for faster checkouts.
* To **access commit history**, use:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

---

### 📌 Summary

| Use Case                 | How to Do It                   |
| ------------------------ | ------------------------------ |
| Checkout current repo    | `uses: actions/checkout@v4`    |
| Checkout specific branch | `with: ref: branch-name`       |
| Access another repo      | `with: repository:` + `token:` |
| Need full git history    | `with: fetch-depth: 0`         |

---

---

Excellent, Deepak! Let’s now look at one of the **core building blocks** of GitHub Actions:

---

## 🔧 Topic: **Introduction to Actions (in GitHub Actions)**

---

### 📘 1. What Is an “Action”?

An **Action** in GitHub Actions is a **reusable unit of code** used to perform a specific task in your CI/CD pipeline.

You can think of an **Action** like a **function** or **plugin** that can be:

* Created by **GitHub**
* Developed by the **community**
* Written by **your team**

They can be used to:

* Checkout code (`actions/checkout`)
* Setup a programming language environment
* Lint or test code
* Build and push Docker images
* Deploy to cloud

---

### 🧱 2. Types of Actions

| Type                  | Description                                              |
| --------------------- | -------------------------------------------------------- |
| **JavaScript Action** | Runs using Node.js, fastest, works cross-platform        |
| **Docker Action**     | Runs inside a container, great for isolated environments |
| **Composite Action**  | Combines multiple steps in YAML, no scripting needed     |

---

### 💡 3. Real-Life Use Cases

| Scenario                     | Action Example                             |
| ---------------------------- | ------------------------------------------ |
| Checkout repo                | `actions/checkout@v4`                      |
| Setup Node.js environment    | `actions/setup-node@v4`                    |
| Run Terraform fmt & validate | `hashicorp/setup-terraform@v3`             |
| Upload artifacts to GitHub   | `actions/upload-artifact@v4`               |
| Deploy to AWS                | `aws-actions/configure-aws-credentials@v3` |

---

### 🔨 4. Basic Example – Using a Prebuilt Action

```yaml
name: Run Node Lint

on: [push]

jobs:
  node-lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4  # ✅ Checkout source code

      - uses: actions/setup-node@v4  # ✅ Setup Node.js
        with:
          node-version: '20'

      - run: npm install
      - run: npm run lint
```

---

### 📦 5. Using Docker-Based Action

You can use Docker-based actions like this:

```yaml
- uses: addnab/docker-run-action@v3
  with:
    image: alpine
    run: |
      echo "Running inside Alpine!"
```

This action spins up a container and runs the `run:` block inside it.

---

### 🔗 6. Using Actions from a Different Repo

```yaml
- uses: octocat/hello-world-action@v1
```

📌 The format is:

```
<owner>/<repo>@<tag/branch>
```

---

### 🧱 7. Composite Actions (Reusable Multi-Step Actions)

Used when you want to create your own action with multiple internal steps.

📁 File structure:

```
.github/
  actions/
    my-lint-action/
      action.yml
```

📄 `action.yml`

```yaml
name: Lint Code
description: Run ESLint and Prettier

runs:
  using: "composite"
  steps:
    - run: npm install
    - run: npm run lint
```

✅ Then use it like:

```yaml
- uses: ./.github/actions/my-lint-action
```

---

### 🧠 8. Custom JavaScript Action

Here’s a **basic JavaScript action**:

📁 `action.yml`

```yaml
name: Say Hello
runs:
  using: "node16"
  main: "index.js"
```

📄 `index.js`

```js
console.log("Hello from custom JavaScript action!");
```

You can **publish** this to the GitHub Marketplace too!

---

```yaml
name: Simple Action
on: [push]

jobs:
  simple-action:
    runs-on: ubuntu-latest
    steps:
      - name: Simple JS Action
        id: greet
        uses: actions/hello-world-javascript-action@e76147da8e5c81eaf017dede5645551d4b94427b
        with:
          who-to-greet: Ali
      - name: Log Greeting Time
        run: echo "${{ steps.greet.outputs.time }}"
```

### ⚠️ 9. Common Pitfalls

| Mistake                               | Issue                                      |
| ------------------------------------- | ------------------------------------------ |
| Forgetting `@vX` tag                  | Always use versioned actions, not branches |
| Using relative paths wrongly          | Composite actions must be inside the repo  |
| Misusing Docker image in wrong runner | Docker actions need Linux runners          |

---

### 🛠 10. Pro Tips

* 🔖 Always pin to **major version** like `@v4` (not `@main`) for stability.
* 🧪 Test your custom actions locally using `act` tool.
* 🔄 Reuse logic across repos by **publishing your own action** to the marketplace.

---

### ✅ Summary

| What                    | Example                           |
| ----------------------- | --------------------------------- |
| Use official action     | `actions/checkout@v4`             |
| Use community action    | `docker/login-action@v2`          |
| Create composite action | YAML with multiple steps          |
| Create JS/Docker action | Custom logic with reusable config |

---

#### Links to Action Marketplace

[Actions Marketplace](https://github.com/marketplace?type=actions)

---






