# CI-CD-Pipeline

## Github Action Workflow

```bash
name: CI/CD Pipeline
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm test
      - uses: docker/setup-buildx-action@v2
      - uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/my-app:latest
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v3
      - name: Install Docker Compose
        run: |
          sudo apt-get update
          sudo apt-get install -y docker-compose
      - uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - run: docker-compose pull
      - run: docker-compose -f docker-compose.yml up -d
```

### **CI/CD Pipeline Summary**

This GitHub Actions workflow automates the **build**, **test**, and **deployment** process for a Node.js application using Docker.

### **Workflow Triggers**
- **Trigger**: Runs whenever there is a `push` to the `main` branch.


### **Jobs**

#### **1. Build Job**
- **Runs on**: `ubuntu-latest` (GitHub-hosted runner).
- **Purpose**: Build, test, and package the application into a Docker image.
- **Steps**:
   1. **Checkout Code**: Pulls the latest code from the repository.
   2. **Setup Node.js**: Installs Node.js version `18` and caches dependencies for faster builds.
   3. **Install Dependencies**: Installs project dependencies using `npm ci`.
   4. **Run Tests**: Executes unit tests using `npm test`.
   5. **Setup Docker Buildx**: Enables advanced Docker builds using Buildx.
   6. **Login to Docker Hub**: Authenticates to Docker Hub using secrets:
      - `DOCKER_USERNAME`  
      - `DOCKER_PASSWORD`
   7. **Build and Push Docker Image**:
      - Builds the Docker image using the current context (`.`).
      - Tags the image as `DOCKER_USERNAME/my-app:latest`.
      - Pushes the image to Docker Hub.

#### **2. Deploy Job**
- **Runs on**: `ubuntu-latest` (GitHub-hosted runner).
- **Dependencies**: Requires the `build` job to complete successfully.
- **Purpose**: Deploy the Docker image using Docker Compose.
- **Steps**:
   1. **Checkout Code**: Pulls the latest code from the repository.
   2. **Install Docker Compose**: Installs `docker-compose` on the runner.
   3. **Login to Docker Hub**: Authenticates to Docker Hub.
   4. **Pull Docker Image**: Pulls the updated Docker image from Docker Hub.
   5. **Deploy with Docker Compose**:
      - Starts or updates the services defined in the `docker-compose.yml` file.
      - Runs the containers in detached mode (`-d`).

