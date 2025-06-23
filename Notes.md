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

Let me know which topic you'd like to dive into next! (e.g., Docker integration, custom actions, environments). 🚀
