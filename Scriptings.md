Here's a comprehensive **GitHub Actions CI/CD pipeline** YAML file for a **Java application built with Maven**, incorporating the following tools:

* **SonarQube**: For static code analysis
* **OWASP Dependency-Check**: For checking vulnerable dependencies
* **Trivy**: For container vulnerability scanning
* **Docker**: For image building
* **Amazon ECR and Docker Hub**: For pushing Docker images
* **Nexus**: For publishing Maven artifacts
* Security and cache best practices

---

### 📄 `.github/workflows/cicd-pipeline.yml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  IMAGE_NAME: my-java-app
  REGISTRY: ghcr.io
  ECR_REPOSITORY: my-ecr-repo
  DOCKERHUB_REPO: mydockerhubuser/my-java-app
  SONAR_HOST_URL: https://sonarqube.example.com

jobs:
  build:
    name: Build & Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'

      - name: Run OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@v1.0.1
        with:
          project: 'Java App'
          path: '.'
          format: 'HTML'

      - name: SonarQube Scan
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: |
          mvn sonar:sonar \
            -Dsonar.projectKey=my-java-app \
            -Dsonar.host.url=${{ env.SONAR_HOST_URL }} \
            -Dsonar.login=${{ env.SONAR_TOKEN }}

      - name: Maven Build & Package
        run: mvn clean package -DskipTests

      - name: Upload Artifact to Nexus
        run: mvn deploy -DskipTests
        env:
          MAVEN_USERNAME: ${{ secrets.NEXUS_USERNAME }}
          MAVEN_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}

  docker:
    name: Build & Push Docker Images
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: |
          docker build -t $IMAGE_NAME:latest .

      - name: Scan Docker image with Trivy
        uses: aquasecurity/trivy-action@v0.16.0
        with:
          image-ref: $IMAGE_NAME:latest
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          vuln-type: 'os,library'

      - name: Tag and Push to DockerHub
        run: |
          docker tag $IMAGE_NAME:latest ${{ env.DOCKERHUB_REPO }}:latest
          docker push ${{ env.DOCKERHUB_REPO }}:latest

      - name: Tag and Push to Amazon ECR
        run: |
          ECR_URI=$(aws ecr describe-repositories --repository-names $ECR_REPOSITORY --query "repositories[0].repositoryUri" --output text)
          docker tag $IMAGE_NAME:latest $ECR_URI:latest
          docker push $ECR_URI:latest
```

---

### 🔐 Secrets You Need in GitHub

Set these in **Repository Settings > Secrets and variables > Actions > Secrets**:

* `SONAR_TOKEN`: SonarQube login token
* `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`
* `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
* `NEXUS_USERNAME`, `NEXUS_PASSWORD`

---


