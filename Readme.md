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


