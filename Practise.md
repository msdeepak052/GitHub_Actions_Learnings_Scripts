Here’s a comprehensive practice set to help you master **GitHub Actions** — from **Beginner to Advanced** level — with **20 questions per level**, each question including a **hint** to guide your thinking or research.

---

# ✅ Beginner Level (20 Questions)

These focus on core concepts, syntax, and basic operations.

| #  | Question                                                                      | Hint                                             |
| -- | ----------------------------------------------------------------------------- | ------------------------------------------------ |
| 1  | What is the purpose of `.github/workflows/` directory?                        | Think of where GitHub looks for workflow files.  |
| 2  | How do you trigger a workflow on every push to any branch?                    | Use the `on:` key.                               |
| 3  | What does `runs-on: ubuntu-latest` specify in a workflow?                     | It's about the environment.                      |
| 4  | How do you define a simple job with a name "Build" that echoes "Hello World"? | Use `jobs:`, `steps:` and `run:`.                |
| 5  | What is the default shell used in GitHub Actions for Ubuntu runners?          | Consider Unix shell behavior.                    |
| 6  | How can you trigger a workflow manually via the UI?                           | Think about `workflow_dispatch`.                 |
| 7  | How do you reference the output of a previous step in the same job?           | Use `id` and `steps.<id>.outputs`.               |
| 8  | How do you access the branch name in GitHub Actions?                          | Check `github.ref` or `github.head_ref`.         |
| 9  | How do you cache dependencies in GitHub Actions?                              | Look into `actions/cache@v3`.                    |
| 10 | How do you specify matrix builds in a workflow?                               | Try `strategy: matrix:`.                         |
| 11 | How do you use an environment variable in a `run:` script?                    | Use `$VAR_NAME` syntax.                          |
| 12 | What is the purpose of `uses:` in a step?                                     | Think about calling an action.                   |
| 13 | How do you check out the code in a GitHub Action workflow?                    | Look at `actions/checkout`.                      |
| 14 | How do you prevent a job from running unless a file in `src/` is changed?     | Use `paths:` filter.                             |
| 15 | What does `continue-on-error: true` do?                                       | It relates to error handling.                    |
| 16 | How do you reference a secret stored in GitHub repository settings?           | Think `secrets.NAME`.                            |
| 17 | How do you upload an artifact from a job?                                     | Try `actions/upload-artifact`.                   |
| 18 | Can workflows call other workflows? If yes, how?                              | Look into `workflow_call`.                       |
| 19 | What does `if: github.actor == 'bot-user'` do?                                | Conditional execution.                           |
| 20 | How do you use reusable workflows?                                            | `uses: <repo>/.github/workflows/<name>.yml@ref`. |

---

# ⚙️ Intermediate Level (20 Questions)

Covers CI/CD pipelines, expressions, reusable workflows, and artifacts.

| #  | Question                                                                                      | Hint                                                      |
| -- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| 1  | How do you define reusable workflow inputs and outputs?                                       | Try `inputs:` and `outputs:`.                             |
| 2  | How do you create a reusable workflow that only runs on pull requests to `main`?              | Combine `workflow_call` and `on:` logic.                  |
| 3  | What does `${{ github.event.pull_request.title }}` access?                                    | It's from the event payload.                              |
| 4  | How do you pass a job’s output to another job?                                                | Define `outputs:` and use `needs:`.                       |
| 5  | How do you prevent concurrent runs of the same workflow?                                      | Use `concurrency:`.                                       |
| 6  | How do you store a deployment environment in a workflow?                                      | Use `environment:`.                                       |
| 7  | How can you define and use a composite action?                                                | Learn about `action.yml` and multiple steps.              |
| 8  | What is the difference between `needs:` and `depends-on:` in GitHub Actions?                  | One exists, one doesn't.                                  |
| 9  | How do you run a job only when the previous job failed?                                       | Use `if: failure()`.                                      |
| 10 | What is the use of `jobs.<job_id>.continue-on-error`?                                         | It modifies default behavior on failure.                  |
| 11 | How do you filter events using branch or tag names?                                           | Look at `branches:` and `tags:`.                          |
| 12 | How do you use GitHub Actions to deploy a static site to GitHub Pages?                        | `actions/upload-pages-artifact` and `deploy-pages`.       |
| 13 | How do you debug workflow steps during development?                                           | Enable `ACTIONS_STEP_DEBUG`.                              |
| 14 | How do you set a custom status badge for a GitHub Actions workflow?                           | Construct URL based on repo/workflow name.                |
| 15 | How do you store workflow runtime secrets securely?                                           | Use GitHub repo `Secrets`.                                |
| 16 | What is the benefit of using matrix builds in testing?                                        | Think multi-env testing.                                  |
| 17 | How do you use `jobs.<job_id>.outputs` in another job?                                        | Combine `needs:` and `${{ needs.<job>.outputs.<name> }}`. |
| 18 | How can you test Docker containers inside GitHub Actions?                                     | Use `services:` or `docker` CLI.                          |
| 19 | How do you trigger one workflow after the successful run of another workflow in another repo? | Use `workflow_dispatch` + `github-script`.                |
| 20 | What does `actions/github-script` allow you to do?                                            | Use JavaScript with `github` context.                     |

---

# 🔥 Advanced Level (20 Questions)

Focuses on self-hosted runners, custom actions, publishing actions, deployment, and advanced automation.

| #  | Question                                                                      | Hint                                                    |
| -- | ----------------------------------------------------------------------------- | ------------------------------------------------------- |
| 1  | How do you configure and register a self-hosted runner on a Linux VM?         | Look into GitHub's Runner CLI.                          |
| 2  | How can you restrict workflow execution to specific runners by labels?        | Use `runs-on: [self-hosted, label]`.                    |
| 3  | How do you monitor the health and status of self-hosted runners?              | GitHub UI or REST API.                                  |
| 4  | How do you publish a custom JavaScript action to GitHub Marketplace?          | `action.yml` + release + tag.                           |
| 5  | What files are required to create a JavaScript GitHub Action?                 | Look into `action.yml`, `index.js`.                     |
| 6  | How do you cache Docker layers between builds?                                | Use `actions/cache` with `~/.docker`.                   |
| 7  | How do you publish a Docker container GitHub Action?                          | `Dockerfile` + `action.yml`.                            |
| 8  | How do you deploy an app to AWS using GitHub Actions?                         | Use `aws-actions/configure-aws-credentials`.            |
| 9  | How can you encrypt deployment credentials in GitHub Actions?                 | Use encrypted repo secrets.                             |
| 10 | How do you prevent secrets from leaking in logs?                              | Redaction is automatic, but avoid `echo $SECRET`.       |
| 11 | How do you use environment protection rules in workflows?                     | Set `environment:` and require reviewers.               |
| 12 | How do you use OIDC (OpenID Connect) to access AWS from GitHub Actions?       | Look into GitHub OIDC token integration.                |
| 13 | How do you dynamically generate a matrix based on a JSON list?                | Use `fromJSON()` in expressions.                        |
| 14 | How can you poll another workflow’s status before continuing?                 | Use `github-script` or GitHub API.                      |
| 15 | How can you share artifacts across jobs and workflows?                        | `upload-artifact` + `download-artifact`.                |
| 16 | How do you build and push a Docker image to ECR using GitHub Actions?         | Use `aws-actions` and `docker build/push`.              |
| 17 | How do you build and test a mono-repo using conditional matrix jobs?          | Combine `matrix` with filters.                          |
| 18 | How can you trigger workflows across repositories in different organizations? | Use GitHub token and `workflow_dispatch`.               |
| 19 | How do you implement Canary Deployments using GitHub Actions?                 | Consider phased rollouts with flags.                    |
| 20 | How do you secure GitHub Actions workflows from unauthorized tampering?       | Use `CODEOWNERS`, review rules, and branch protections. |

---


* ✅ Beginner (20 hands-on workflow tasks)
* ⚙️ Intermediate (20 practice scenarios)
* 🔥 Advanced (20 challenging workflow exercises)

Each problem comes with a **description** and a **hint** to guide implementation. You can run most of them in a test repo.

---

## ✅ Beginner Practice Set (20 Workflow Problems)

| #  | Task                                                                                   | Hint                                         |
| -- | -------------------------------------------------------------------------------------- | -------------------------------------------- |
| 1  | Create a workflow that runs on every push to `main` and prints "Hello GitHub Actions". | Use `on: push` with `branches`.              |
| 2  | Set up a workflow to trigger manually from the Actions tab.                            | Use `workflow_dispatch`.                     |
| 3  | Create a job that runs on `ubuntu-latest` and lists all files in the repo.             | Use `ls -la`.                                |
| 4  | Set an environment variable `GREETING=Hi` and print it.                                | Use `env:` at the step or job level.         |
| 5  | Checkout code and run `npm install` (assume package.json exists).                      | Use `actions/checkout` and `run:`.           |
| 6  | Add a second job called `test` that depends on a `build` job.                          | Use `needs:`.                                |
| 7  | Run a job only when the actor is not `dependabot`.                                     | Use `if: github.actor != 'dependabot[bot]'`. |
| 8  | Use a matrix build to test on Node.js versions 14 and 16.                              | Use `strategy.matrix.node-version`.          |
| 9  | Cache `node_modules` using `actions/cache`.                                            | Use correct `key` and `path`.                |
| 10 | Print the branch name using the GitHub context.                                        | Use `${{ github.ref }}` or `steps`.          |
| 11 | Create a workflow that runs on pull requests only.                                     | `on: pull_request`.                          |
| 12 | Use `actions/upload-artifact` to store build output.                                   | Store a `.zip` or build folder.              |
| 13 | Download an artifact in another job.                                                   | Use `actions/download-artifact`.             |
| 14 | Reference a secret called `MY_SECRET` in your workflow.                                | Use `${{ secrets.MY_SECRET }}`.              |
| 15 | Fail a step intentionally and mark it to not fail the workflow.                        | Use `continue-on-error: true`.               |
| 16 | Create a reusable workflow with an input called `name`.                                | Use `workflow_call` and `inputs`.            |
| 17 | Call a reusable workflow from another using `uses:`.                                   | Use local `.github/workflows/`.              |
| 18 | Set a job output `message` as "done" and use it in another job.                        | Use `outputs:` and `needs`.                  |
| 19 | Create a badge for your workflow in the `README.md`.                                   | Use Actions badge markdown syntax.           |
| 20 | Schedule your workflow to run every day at 3am UTC.                                    | Use `on: schedule` with cron syntax.         |

---

## ⚙️ Intermediate Practice Set (20 Problems)

| #  | Task                                                                      | Hint                                             |
| -- | ------------------------------------------------------------------------- | ------------------------------------------------ |
| 1  | Trigger workflow only if a file in `src/` changes.                        | Use `on.push.paths`.                             |
| 2  | Use matrix to test across OS: `ubuntu`, `macos`, and `windows`.           | `matrix.os` in `runs-on`.                        |
| 3  | Store and retrieve job output between steps.                              | Use `id:` and `outputs`.                         |
| 4  | Set job-level environment variable and use in multiple steps.             | Use `env:` under `jobs`.                         |
| 5  | Use custom `GITHUB_TOKEN` to comment on a PR.                             | Use `github-script` or REST API.                 |
| 6  | Create a composite action that lints and formats code.                    | Use `action.yml` + shell steps.                  |
| 7  | Upload build artifacts only when on `main` branch.                        | `if: github.ref == 'refs/heads/main'`.           |
| 8  | Build and push a Docker image to DockerHub.                               | Use Docker CLI in steps.                         |
| 9  | Build and push Docker image to AWS ECR.                                   | Use `aws-actions/configure-aws-credentials`.     |
| 10 | Set `concurrency` group to avoid duplicate runs.                          | Use `concurrency: group`.                        |
| 11 | Add manual approval step using `environments`.                            | Use `environment: staging`.                      |
| 12 | Deploy to GitHub Pages using `deploy-pages`.                              | Use `actions/deploy-pages`.                      |
| 13 | Use `jobs.<job>.if` to skip build if commit message contains `[skip ci]`. | Use expression with `contains()`.                |
| 14 | Send Slack notification after deployment.                                 | Use Slack webhook.                               |
| 15 | Run tests in parallel using matrix and collect results.                   | Use matrix + artifact upload.                    |
| 16 | Retry a failed step up to 3 times.                                        | Use `retry` pattern via script or action.        |
| 17 | Use `.env` file and load variables into the job.                          | `dotenv` parser or `source` it in a `bash` step. |
| 18 | Use `strategy.fail-fast: false` to continue matrix jobs.                  | Helpful in test suites.                          |
| 19 | Trigger workflow only on push to `src/**.js` files.                       | Use `on.push.paths`.                             |
| 20 | Use GitHub Actions to deploy a Node app to Heroku.                        | Use Heroku CLI and API key in secrets.           |

---

## 🔥 Advanced Workflow Challenges (20 Problems)

| #  | Task                                                                             | Hint                                                               |
| -- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 1  | Setup and register a self-hosted runner on Ubuntu.                               | Use `./config.sh` and `./run.sh`.                                  |
| 2  | Create a self-hosted runner group and restrict workflows to it.                  | Use `runs-on: [self-hosted, <label>]`.                             |
| 3  | Use OIDC with AWS to assume IAM role from GitHub Actions.                        | Use `aws-actions/configure-aws-credentials` with `role-to-assume`. |
| 4  | Trigger a workflow in another repo using the REST API.                           | Use `actions/github-script`.                                       |
| 5  | Build a Docker-based GitHub Action and publish it to Marketplace.                | Requires `Dockerfile` and `action.yml`.                            |
| 6  | Create a JavaScript GitHub Action with Octokit to list issues.                   | Use `@actions/core`, `@actions/github`.                            |
| 7  | Implement approval gates using `environments` and protected reviewers.           | Repo settings required.                                            |
| 8  | Use `fromJSON()` to dynamically build a matrix strategy.                         | Useful for dynamic values.                                         |
| 9  | Create a reusable workflow that triggers only from `dev-*` branches.             | Combine `workflow_call` and `on.push.branches`.                    |
| 10 | Poll the status of another repo’s workflow until completion.                     | Use GitHub REST API.                                               |
| 11 | Store Terraform plan output as artifact and use in apply job.                    | Use `upload-artifact`.                                             |
| 12 | Create a pipeline to scan Docker images with Trivy and block on vulnerabilities. | Trivy GitHub Action.                                               |
| 13 | Deploy to Kubernetes cluster using `kubectl` from GitHub Actions.                | Requires kubeconfig in secrets.                                    |
| 14 | Build and push a Helm chart to OCI registry using GitHub Actions.                | Use `helm` CLI in the runner.                                      |
| 15 | Use secrets stored in AWS Secrets Manager using GitHub Actions.                  | Pull via `aws secretsmanager`.                                     |
| 16 | Implement canary deployments with GitHub Actions and feature flags.              | Split traffic using deployment scripts.                            |
| 17 | Run performance tests after deployment and send report.                          | Use Apache Benchmark or k6.                                        |
| 18 | Set up a workflow to trigger on a tag that matches `v*.*.*`.                     | Use `on.push.tags`.                                                |
| 19 | Create a GitHub Action that lints YAML and JSON files only on PRs.               | Combine filters and reusable action.                               |
| 20 | Use concurrency + cancellation to cancel stale deployments.                      | Use `concurrency: deploy-${{ github.ref }}`.                       |

---


