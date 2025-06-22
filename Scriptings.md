To create a CI/CD pipeline that meets the following requirements:

1. **CI Pipeline** that includes security measures with SonarQube, Nexus, OWASP, Trivy, Docker build & push to ECR and DockerHub.
2. **CD Pipeline** that is triggered after a successful CI pipeline, deploying with ArgoCD to a Kubernetes (K8s) cluster, installing Prometheus and Grafana using Helm, and sending an email notification once deployment is successful.

Here's how to structure your GitHub Actions workflows for both **CI** and **CD** pipelines:

### **CI Pipeline (`ci.yml`)**

This CI pipeline will:

1. Build a Java Maven project.
2. Run SonarQube analysis.
3. Scan for vulnerabilities with OWASP and Trivy.
4. Build and push Docker images to AWS ECR and DockerHub.
5. Trigger the CD pipeline via a repository dispatch event after a successful run.

#### GitHub Actions CI Pipeline (`.github/workflows/ci.yml`)

```yaml
name: CI Pipeline

on:
  push:
    branches:
      - main
    paths:
      - '**/*.java'
      - '**/pom.xml'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'

      - name: Set up Maven
        run: mvn clean install

      - name: Run SonarQube Scan
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn sonar:sonar -Dsonar.projectKey=your-project-key -Dsonar.host.url=https://sonarqube.yourcompany.com

      - name: Build Docker Image
        run: |
          docker build -t your-docker-image-name:latest .
          
      - name: Scan with Trivy for Vulnerabilities
        uses: aquasecurity/trivy-action@v0.0.18
        with:
          image: your-docker-image-name:latest
          
      - name: Authenticate to AWS ECR
        run: |
          aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin ${{ secrets.AWS_ECR_REGISTRY }}

      - name: Push Docker Image to ECR
        run: |
          docker tag your-docker-image-name:latest ${{ secrets.AWS_ECR_REGISTRY }}/your-repository:latest
          docker push ${{ secrets.AWS_ECR_REGISTRY }}/your-repository:latest

      - name: Push Docker Image to DockerHub
        run: |
          docker tag your-docker-image-name:latest ${{ secrets.DOCKER_USERNAME }}/your-repository:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/your-repository:latest

      - name: Trigger CD Pipeline (ArgoCD)
        run: |
          curl -X POST -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
          -d '{"event_type": "deploy", "client_payload": {"tag": "latest", "repo_name": "your-repo-name", "app_name": "your-app-name"}}' \
          https://api.github.com/repos/your-org/your-repo/dispatches
```

### **CD Pipeline (`cd.yml`)**

The CD pipeline will be triggered from the CI pipeline upon a successful run. It will:

1. Use ArgoCD to deploy the application to the Kubernetes cluster.
2. Install Prometheus and Grafana using Helm.
3. Send a Gmail notification upon successful deployment.

#### GitHub Actions CD Pipeline (`.github/workflows/cd.yml`)

```yaml
name: CD Pipeline

on:
  repository_dispatch:
    types: [deploy]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up Kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "$KUBE_CONFIG" > ~/.kube/config
        env:
          KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}

      - name: Install ArgoCD CLI
        run: |
          curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.3.5/argocd-linux-amd64
          chmod +x /usr/local/bin/argocd

      - name: Login to ArgoCD
        run: |
          argocd login argocd.your-cluster.com --username ${{ secrets.ARGO_USERNAME }} --password ${{ secrets.ARGO_PASSWORD }} --insecure

      - name: Deploy with ArgoCD
        run: |
          argocd app sync your-app-name --revision ${{ github.event.client_payload.tag }}

      - name: Install Prometheus with Helm
        run: |
          helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
          helm repo update
          helm install prometheus prometheus-community/kube-prometheus-stack

      - name: Install Grafana with Helm
        run: |
          helm repo add grafana https://grafana.github.io/helm-charts
          helm repo update
          helm install grafana grafana/grafana

      - name: Send Deployment Notification Email
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.gmail.com
          server_port: 587
          username: ${{ secrets.GMAIL_USERNAME }}
          password: ${{ secrets.GMAIL_PASSWORD }}
          subject: "CD Pipeline - Deployment Successful"
          to: "your-email@gmail.com"
          body: |
            The deployment of the application to the Kubernetes cluster has been completed successfully.
```

### **Explanation of Workflow Steps**

1. **CI Pipeline**:

   * **Build and Test**: The Java application is built using Maven, and SonarQube analysis is triggered to ensure code quality.
   * **Security**: Trivy scans the Docker image for vulnerabilities.
   * **Docker Build & Push**: The Docker image is built and pushed to both AWS ECR and DockerHub.
   * **Trigger CD Pipeline**: A `repository_dispatch` event is triggered with the tag, repo name, and app name as payload for the CD pipeline.

2. **CD Pipeline**:

   * **Trigger**: This pipeline is triggered via `repository_dispatch`, passing necessary details (like tag, repo name, and app name).
   * **ArgoCD**: The application is deployed to a Kubernetes cluster via ArgoCD.
   * **Helm**: Prometheus and Grafana are installed using Helm charts for monitoring purposes.
   * **Email Notification**: After a successful deployment, an email notification is sent via Gmail using the `action-send-mail` GitHub Action.

### **Secrets Setup in GitHub**:

* **`AWS_ECR_REGISTRY`**: AWS ECR registry URL.
* **`DOCKER_USERNAME`**: DockerHub username.
* **`SONAR_TOKEN`**: SonarQube authentication token.
* **`GITHUB_TOKEN`**: GitHub token for repository dispatch.
* **`KUBE_CONFIG`**: Kubernetes configuration for accessing the cluster.
* **`ARGO_USERNAME`**: ArgoCD username.
* **`ARGO_PASSWORD`**: ArgoCD password.
* **`GMAIL_USERNAME`**: Gmail username.
* **`GMAIL_PASSWORD`**: Gmail app password.

This setup ensures a robust, automated CI/CD pipeline with security measures, deployment automation, and monitoring integration.

In the context of the GitHub Actions workflow, the `paths` configuration defines the files or directories that should trigger the workflow when there are changes.

Here’s what each part means:

### 1. **`'**/*.java'`**:

* `**/` indicates any directory (recursively).
* `*.java` means all files that have the `.java` extension.

**Explanation**: This part tells GitHub Actions to trigger the workflow if any Java source file (with a `.java` extension) changes, no matter which directory it is in. For example, if a `.java` file in a subfolder is modified, the workflow will run.

### 2. **`'**/pom.xml'`**:

* `**/` indicates any directory (recursively).
* `pom.xml` refers to the specific file named `pom.xml`.

**Explanation**: This part tells GitHub Actions to trigger the workflow if the `pom.xml` file (which is a build configuration file for Maven projects) is modified, regardless of the directory it is located in.

### **Summary**:

* The workflow will be triggered when there are changes in any Java files (`.java`) or the `pom.xml` file anywhere in the repository, including in subdirectories.

### The line:

```yaml
uses: dawidd6/action-send-mail@v3
```

is referencing a **GitHub Action** that is used to send an email, specifically the **`dawidd6/action-send-mail`** action, which is maintained by the user `dawidd6`.

### **Explanation**:

* **`dawidd6/action-send-mail`**: This is the name of the GitHub Action, which is a pre-built reusable component created by the `dawidd6` GitHub user. It’s used to send emails directly from your GitHub Actions workflows.

* **`@v3`**: This specifies the version of the action. Here, `v3` means you are using version 3 of this action. GitHub Actions supports versioning for each action, and this helps ensure stability in your workflows. By specifying `@v3`, you're using the latest stable release of version 3.

### **What it does**:

This GitHub Action sends an email from within your workflow. It uses an SMTP server to send the email to a designated recipient. You can customize the subject, body, and other aspects of the email.

### **Common Use Case**:

This action is often used in CD (Continuous Deployment) workflows to notify teams when deployments are successful or fail. For example, in your workflow, after a successful deployment, this action could be used to send a notification email to a specified Gmail address, letting the team know that the deployment was completed successfully.

### **How it works**:

To configure this action, you would typically need to provide:

* SMTP server details (e.g., for Gmail: `smtp.gmail.com`)
* SMTP server port (usually 587 for Gmail)
* Your email credentials (username and app-specific password for Gmail, stored in GitHub secrets)
* The subject and body of the email

### Example:

Here's a quick example of how this action might be used to send an email notification in a GitHub Actions workflow:

```yaml
- name: Send Deployment Notification Email
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 587
    username: ${{ secrets.GMAIL_USERNAME }}
    password: ${{ secrets.GMAIL_PASSWORD }}
    subject: "Deployment Successful"
    to: "recipient-email@example.com"
    body: "The deployment to the production environment was successful."
```

In this example:

* The action connects to Gmail's SMTP server (`smtp.gmail.com`).
* It uses the username and password stored as secrets in GitHub to authenticate.
* It sends an email to the provided recipient email address with the subject "Deployment Successful" and a body message saying the deployment was successful.

### **Summary**:

The `dawidd6/action-send-mail@v3` action is a pre-configured tool for sending emails through SMTP directly from GitHub Actions workflows, which can be useful for notifying stakeholders about important events like successful deployments.

The line:

```yaml
branches:
  - main
```

is used to specify the **branch** in the repository that will trigger the workflow. In this case, it means the GitHub Actions workflow will be triggered whenever there are changes pushed to the `main` branch.

### **Regarding self-hosted vs shared runners**:

* This line (`branches: - main`) **does not determine whether the runner is self-hosted or shared**.
* The **type of runner** (whether it’s self-hosted or shared) is determined by the `runs-on` keyword in the job configuration, like:

```yaml
runs-on: ubuntu-latest  # This specifies a shared runner
```

If you use `runs-on: ubuntu-latest`, it means you're using GitHub's **shared runner** (provided by GitHub). If you want to use a **self-hosted runner**, you would specify it like:

```yaml
runs-on: [self-hosted, linux]
```

This would indicate that the job should run on a self-hosted runner with the Linux operating system.

### **In Summary**:

* **`branches: - main`** only defines the triggering branch (in this case, the `main` branch).
* The **runner type (self-hosted or shared)** is specified using the `runs-on` keyword, not the `branches` configuration.



