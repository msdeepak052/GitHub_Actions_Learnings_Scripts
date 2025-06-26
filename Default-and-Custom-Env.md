# GitHub Actions Environment Variables: Default and Custom

Here's a comprehensive explanation of environment variables in GitHub Actions with real-world examples, based on the [official documentation](https://docs.github.com/en/actions/how-tos/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#default-environment-variables).


### Common Default Variables

1. `GITHUB_WORKFLOW` - Name of the workflow
2. `GITHUB_RUN_ID` - Unique ID for the run
3. `GITHUB_ACTOR` - Username who triggered the workflow
4. `GITHUB_REPOSITORY` - Repository name (owner/repo)
5. `GITHUB_REF` - Branch or tag ref that triggered the run
6. `GITHUB_SHA` - Commit SHA that triggered the run
7. `GITHUB_WORKSPACE` - Path to the workspace directory

## Custom Environment Variables

You can define your own variables at three levels:
1. **Workflow-level**: For entire workflow
2. **Job-level**: For specific job
3. **Step-level**: For specific step
.

---

## 🔹 What are Environment Variables in GitHub Actions?

Environment variables let you **store and use information** throughout your workflows — like credentials, file paths, flags, or metadata.

---

## ✅ 1. **Default Environment Variables**

These are automatically provided by GitHub and **don’t require any setup**.

📌 Some important ones:

| Variable            | Description                                       |
| ------------------- | ------------------------------------------------- |
| `GITHUB_REPOSITORY` | `owner/repo-name`                                 |
| `GITHUB_REF`        | Git ref (branch/tag) that triggered the workflow  |
| `GITHUB_SHA`        | Commit SHA                                        |
| `GITHUB_ACTOR`      | The user who triggered the workflow               |
| `GITHUB_WORKSPACE`  | The directory where the repo is checked out       |
| `GITHUB_EVENT_NAME` | The name of the event that triggered the workflow |
| `GITHUB_RUN_NUMBER` | Unique run number                                 |
| `GITHUB_JOB`        | Job ID                                            |

📘 Reference: [GitHub Docs – Default Env Vars](https://docs.github.com/en/actions/how-tos/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#default-environment-variables)

---

### 🎯 Real-Life Example: CI Pipeline with Logging and Artifact Info

```yaml
name: Build Docker Image

on:
  push:
    branches: [ main ]

jobs:
  docker-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Show GitHub context
        run: |
          echo "Repository: $GITHUB_REPOSITORY"
          echo "Branch/Tag: $GITHUB_REF"
          echo "Commit SHA: $GITHUB_SHA"
          echo "Triggered by: $GITHUB_ACTOR"
          echo "Run number: $GITHUB_RUN_NUMBER"
          echo "Workspace path: $GITHUB_WORKSPACE"

      - name: Build Docker image
        run: |
          docker build -t myapp:${GITHUB_SHA} .
```

🔍 Use Case:

* CI builds tag Docker images with commit SHA.
* Helpful for traceability in your CI/CD lifecycle.

---

## 🛠️ 2. **Custom Environment Variables**

You can define your own variables:

* **At job level**
* **At step level**
* Or using `env` key globally

---

### 🎯 Real-Life Example: Slack Notification, Docker Tagging, Terraform

```yaml
name: Deploy Infrastructure

on:
  workflow_dispatch:

env:
  ENVIRONMENT: "prod"
  REGION: "ap-south-1"
  SLACK_CHANNEL: "#deployments"

jobs:
  deploy:
    runs-on: ubuntu-latest

    env:
      TF_BACKEND_BUCKET: "terraform-${{ github.repository }}"

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Show deployment context
        run: |
          echo "Deploying to $ENVIRONMENT environment in $REGION"
          echo "Using backend bucket: $TF_BACKEND_BUCKET"

      - name: Run Terraform init
        run: |
          terraform init \
            -backend-config="bucket=$TF_BACKEND_BUCKET"

      - name: Run Terraform apply
        run: terraform apply -auto-approve

      - name: Notify Slack
        run: ./scripts/slack-notify.sh "$SLACK_CHANNEL" "Deployment to $ENVIRONMENT complete."
```

### ✅ Breakdown:

| Variable                                 | Scope        | Purpose                               |
| ---------------------------------------- | ------------ | ------------------------------------- |
| `ENVIRONMENT`, `REGION`, `SLACK_CHANNEL` | Global `env` | Shared across all jobs                |
| `TF_BACKEND_BUCKET`                      | Job-level    | Terraform backend bucket per repo     |
| `$GITHUB_REPOSITORY`                     | Default      | Used to construct dynamic bucket name |

---

## 🔁 Use Both Together

```yaml
env:
  CUSTOM_TAG: "v1.0.${{ github.run_number }}"
```

Then use it in:

```bash
docker build -t myapp:$CUSTOM_TAG .
```

---

## 📝 Summary

| Type                 | Defined by | Use Case                               |
| -------------------- | ---------- | -------------------------------------- |
| **Default Env Vars** | GitHub     | Metadata like commit, actor, repo      |
| **Custom Env Vars**  | You        | Configuration like region, bucket, tag |

---

## 📚 Reference

🔗 [GitHub Docs – Environment Variables](https://docs.github.com/en/actions/how-tos/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#default-environment-variables)

---



## Real-World Examples

### Example 1: Deployment with Environment Variables

```yaml
name: Production Deployment
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'staging'

env:
  AWS_REGION: 'us-east-1'
  DEPLOY_TIMEOUT: '300'

jobs:
  setup:
    runs-on: ubuntu-latest
    env:
      BUILD_DIR: 'dist'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Print workflow info
        run: |
          echo "Workflow: $GITHUB_WORKFLOW"
          echo "Triggered by: $GITHUB_ACTOR"
          echo "Repository: $GITHUB_REPOSITORY"
          echo "Commit: $GITHUB_SHA"
          
      - name: Set deployment environment
        run: |
          echo "DEPLOY_ENV=${{ github.event.inputs.environment }}" >> $GITHUB_ENV
          echo "DEPLOY_TIME=$(date)" >> $GITHUB_ENV
          
      - name: Build application
        env:
          BUILD_MODE: 'production'
        run: |
          ./build.sh --mode $BUILD_MODE --dir $BUILD_DIR
          echo "BUILD_VERSION=$(cat version.txt)" >> $GITHUB_ENV
          
  deploy:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - name: Print deployment info
        run: |
          echo "Deploying $BUILD_VERSION to $DEPLOY_ENV"
          echo "AWS Region: $AWS_REGION"
          echo "Timeout: $DEPLOY_TIMEOUT seconds"
          echo "Scheduled at: $DEPLOY_TIME"
          
      - name: Deploy to AWS
        run: ./deploy.sh --env $DEPLOY_ENV --version $BUILD_VERSION
```

### Example 2: CI Pipeline with Custom Variables

```yaml
name: CI Pipeline
on: [push, pull_request]

env:
  DOCKER_REGISTRY: 'ghcr.io'
  ARTIFACT_DIR: 'artifacts'

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      TEST_REPORTS_DIR: 'test-results'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          
      - name: Install dependencies
        run: pip install -r requirements.txt
        
      - name: Run tests
        env:
          TEST_COVERAGE: 'true'
        run: |
          pytest --junitxml=$TEST_REPORTS_DIR/results.xml
          if [ "$TEST_COVERAGE" = "true" ]; then
            coverage xml -o $TEST_REPORTS_DIR/coverage.xml
          fi
          
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: test-reports
          path: ${{ env.TEST_REPORTS_DIR }}
          
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Set image tag
        id: set-tag
        run: |
          if [ "$GITHUB_EVENT_NAME" = "pull_request" ]; then
            echo "IMAGE_TAG=pr-$GITHUB_REF_NAME" >> $GITHUB_OUTPUT
          else
            echo "IMAGE_TAG=$GITHUB_SHA" >> $GITHUB_OUTPUT
          fi
          
      - name: Build Docker image
        run: |
          docker build -t $DOCKER_REGISTRY/$GITHUB_REPOSITORY:${{ steps.set-tag.outputs.IMAGE_TAG }} .
          
      - name: Log build info
        run: |
          echo "Built image: $DOCKER_REGISTRY/$GITHUB_REPOSITORY:${{ steps.set-tag.outputs.IMAGE_TAG }}"
          echo "Triggered by: $GITHUB_ACTOR"
          echo "Event: $GITHUB_EVENT_NAME"
```

### Example 3: Multi-Environment Configuration

```yaml
name: Environment Configuration Demo
on: [push]

env:
  COMPANY_NAME: 'Acme Corp'
  LOG_LEVEL: 'INFO'

jobs:
  staging:
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: 'staging'
      API_URL: 'https://api.staging.example.com'
    steps:
      - name: Print configuration
        run: |
          echo "Company: $COMPANY_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "API: $API_URL"
          echo "Log Level: $LOG_LEVEL"
          
      - name: Deploy to staging
        env:
          DEPLOY_KEY: ${{ secrets.STAGING_DEPLOY_KEY }}
        run: ./deploy.sh --env $ENVIRONMENT --key $DEPLOY_KEY
        
  production:
    needs: staging
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: 'production'
      API_URL: 'https://api.example.com'
    steps:
      - name: Print configuration
        run: |
          echo "Company: $COMPANY_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "API: $API_URL"
          echo "Log Level: $LOG_LEVEL"
          
      - name: Approve production deployment
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.actions.createWorkflowDispatch({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'production-deploy.yml',
              ref: 'main',
            })
```

## Key Concepts

1. **Variable Scope**:
   - Workflow-level: Available to all jobs
   - Job-level: Available to all steps in that job
   - Step-level: Only available to that step

2. **Setting Variables**:
   - Static: Defined in YAML with `env:`
   - Dynamic: Set during execution using `echo "VAR=value" >> $GITHUB_ENV`

3. **Best Practices**:
   - Use secrets for sensitive data (`${{ secrets.MY_SECRET }}`)
   - Use descriptive variable names
   - Group related variables together
   - Document important variables in comments

These examples demonstrate practical uses of environment variables in CI/CD pipelines, testing workflows, and multi-environment deployments.
