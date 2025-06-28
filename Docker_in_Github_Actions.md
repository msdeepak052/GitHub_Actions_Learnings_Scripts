# Docker in Github Actions
Here's a comprehensive guide to running GitHub Actions jobs in Docker containers with **Java/Maven** examples:

---

### **1. Running Jobs in Docker Containers**
GitHub Actions allows you to run jobs in custom Docker containers instead of the host runner. This ensures consistent environments.

#### **Key Benefits:**
- Isolated dependencies
- Consistent build environments
- Custom tooling pre-installed

---

### **2. Java/Maven Docker Example**
This example builds a Java project with Maven inside a Docker container:

#### **`.github/workflows/maven-build.yml`**
```yaml
name: Java CI with Docker
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: maven:3.8.6-openjdk-17  # Official Maven+JDK image
      volumes:
        - /home/runner/.m2:/root/.m2  # Cache Maven local repo
      options: --network host  # Required for some tests

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Build with Maven
      run: mvn -B package --file pom.xml

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: target
        path: target/*.jar
```

---

### **3. Key Container Configuration Options**

| Parameter       | Description                                                                 | Example                          |
|-----------------|-----------------------------------------------------------------------------|----------------------------------|
| `container.image` | Docker image to use                                                      | `maven:3.8.6-openjdk-17`         |
| `container.credentials` | Registry credentials (if using private images)                       | `{username: ${{ secrets.DOCKER_USER }}, password: ${{ secrets.DOCKER_PASS }}` |
| `container.volumes` | Host directories to mount                                               | `/host/path:/container/path`     |
| `container.env` | Environment variables for container                                      | `{ MAVEN_OPTS: "-Xmx2g" }`       |
| `container.ports` | Ports to expose                                                        | `[8080, 5432]`                   |
| `container.options` | Additional Docker options (like `--network`)                           | `--network host`                 |

---

### **4. Multi-Container Example (Java + Database)**
For integration tests requiring a database:

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: --health-cmd "pg_isready" --health-interval 10s --health-timeout 5s --health-retries 5

    container:
      image: maven:3.8.6-openjdk-17

    steps:
    - uses: actions/checkout@v4
    - run: mvn -B verify -Dspring.datasource.url=jdbc:postgresql://postgres:5432/postgres
```

---

### **5. Caching Maven Dependencies**
Optimize builds by caching `.m2` directory:

```yaml
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2-
```

---

### **6. Official References**
1. [GitHub Docs: Running Jobs in Containers](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)
2. [Official Maven Docker Images](https://hub.docker.com/_/maven)
3. [Docker Container Action Syntax](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runs-for-docker-container-actions)

---

### **7. Best Practices**
1. **Use Official Images**: Prefer verified images like `maven:3.8.6-openjdk-17`
2. **Pin Versions**: Avoid `latest` tags for reproducibility
3. **Cache Dependencies**: Essential for Java/Maven builds
4. **Clean Up**: Add `mvn clean` if disk space is limited
5. **Health Checks**: For dependent services (like databases)

Would you like me to elaborate on any specific aspect?
