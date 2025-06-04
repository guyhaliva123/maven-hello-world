# CI/CD Pipeline Setup Instructions

## Prerequisites

1. **GitHub Repository**: Ensure your code is in a GitHub repository
2. **Docker Hub Account**: Create an account at https://hub.docker.com
3. **Docker Desktop**: Required for local development and containerization
   - **Download**: https://www.docker.com/products/docker-desktop/
   - **Installation**: Follow the installer for your operating system:
     - **Windows**: Windows 10 64-bit or later, WSL 2 enabled
     - **macOS**: macOS 10.15 or later
     - **Linux**: 64-bit distribution with kernel 3.10+
   - **Post-Installation**: Start Docker Desktop and ensure it's running (check system tray/menu bar)
4. **Kubernetes Setup in Docker Desktop**:
   - Open **Docker Desktop** application
   - Navigate to **Settings** (gear icon) → **Kubernetes** tab
   - Check **"Enable Kubernetes"** checkbox
   - Click **"Apply & Restart"** button
   - Wait for Kubernetes to start (you'll see a green indicator in the bottom-left corner)
   - **Verify installation**:
     ```bash
     kubectl cluster-info
     kubectl get nodes
     ```
   - **Troubleshooting**: If Kubernetes fails to start, try:
     - Reset Kubernetes: Settings → Kubernetes → Reset Kubernetes Cluster
     - Ensure sufficient resources: Settings → Resources (minimum 4GB RAM, 2 CPUs)
5. **Java 17**: Application requires JDK 13+ (pipeline uses JDK 17)
   - **Download**: https://adoptium.net/temurin/releases/
   - **Verify**: `java -version`
6. **Helm CLI** (for Kubernetes deployment):
   - **Download**: https://helm.sh/docs/intro/install/
   - **Verify installation**: `helm version`
7. **GitHub Secrets**: Configure the following secrets in your GitHub repository

## GitHub Secrets Configuration

Go to your GitHub repository → Settings → Secrets and variables → Actions

Add these secrets:

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password or access token

## Pipeline Features

### ✅ What the Pipeline Does:

1. **Java 17 Support**:

   - Uses JDK 17 (compatible with your JDK 13+ requirement)
   - Modern JVM optimizations for containers

2. **Automatic Version Management**:

   - Increments patch version automatically on main branch
   - Updates pom.xml with new version

3. **Code Compilation**:

   - Compiles Java code using Maven
   - Uses modern compiler features

4. **Artifact Creation**:

   - Packages code into JAR file
   - Uploads artifact for download

5. **Docker Image Creation**:

   - Builds Docker image with Eclipse Temurin 17
   - Runs as non-root user for security
   - Includes health checks and JVM optimizations

6. **Docker Hub Push**:

   - Pushes image with version tag and 'latest' tag
   - Multi-architecture support (amd64, arm64)

7. **Automated Testing**:
   - Runs unit tests before building
   - Caches Maven dependencies for faster builds

## Local Development

### Automated Build Scripts (Recommended):

#### **Setup Scripts:**

Create these automation scripts in your project root to handle version detection automatically:

**1. Create `build-docker.sh` for local Docker builds:**

```bash
cat > build-docker.sh << 'EOF'
#!/bin/bash

set -e  # Exit on any error

echo "🔄 Auto-Building Docker Container"
echo "================================"

# Navigate to Maven project
cd myapp

# Extract version from pom.xml
echo "🔍 Extracting version from pom.xml..."
VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
echo "📦 Detected version: $VERSION"

# Build Maven project
echo "🏗️  Building Maven project..."
mvn clean package

# Build Docker image with auto-detected version
echo "🐳 Building Docker image with version $VERSION..."
docker build --build-arg JAR_VERSION=$VERSION -t guyhaliva/myapp:$VERSION -t guyhaliva/myapp:latest .

# Go back to root
cd ..

echo "✅ Build complete!"
echo "🏷️  Image tags created:"
echo "   - guyhaliva/myapp:$VERSION"
echo "   - guyhaliva/myapp:latest"

echo ""
echo "🚀 To run the container:"
echo "   docker run -p 8080:8080 guyhaliva/myapp:latest"
EOF
```

**2. Create `compose-build.sh` for Docker Compose:**

```bash
cat > compose-build.sh << 'EOF'
#!/bin/bash

set -e

echo "🔄 Docker Compose with Auto Version"
echo "==================================="

# Get version and export it
cd myapp
export JAR_VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
echo "📦 Version: $JAR_VERSION"

# Build Maven project
mvn clean package
cd ..

# Create/update .env file for docker-compose
echo "JAR_VERSION=$JAR_VERSION" > .env

# Run docker-compose
echo "🐳 Running Docker Compose..."
docker compose up --build
EOF
```

**3. Make scripts executable:**

```bash
chmod +x build-docker.sh
chmod +x compose-build.sh
```

#### **Usage:**

**For local Docker development:**

```bash
# Run the automated build script
./build-docker.sh

# Then run the container
docker run -p 8080:8080 guyhaliva/myapp:latest

# View logs
docker logs -f CONTAINER_NAME
```

**For Docker Compose development:**

```bash
# Run the automated compose script
./compose-build.sh

# Or run in background
./compose-build.sh &

# View logs
docker compose logs -f myapp

# Stop when done
docker compose down
```

### Manual Build Process (Alternative):

If you prefer manual control over the build process:

```bash
# Ensure you have JDK 13+ installed
java -version

# Build the project
cd myapp
mvn clean package

# Run the JAR directly
java -jar target/myapp-*.jar

# Or build Docker manually
VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
docker build --build-arg JAR_VERSION=$VERSION -t guyhaliva/myapp:$VERSION .
docker run --rm -p 8080:8080 guyhaliva/myapp:$VERSION
```

### Test the Pipeline Locally:

```bash
# Using automated script (recommended)
./build-docker.sh

# Or manual testing
cd myapp
mvn clean compile
mvn clean package

# Test Docker build with version
VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
docker build --build-arg JAR_VERSION=$VERSION -t guyhaliva/myapp:$VERSION .
docker tag guyhaliva/myapp:$VERSION guyhaliva/myapp:latest

# Run Docker container with port mapping
docker run --rm -p 8080:8080 guyhaliva/myapp:latest

# Test with logs (run in background)
docker run -d --name myapp-test -p 8080:8080 guyhaliva/myapp:latest
docker logs -f myapp-test
# Clean up: docker stop myapp-test && docker rm myapp-test
```

### Development Workflow Scripts:

#### **Quick Development Commands:**

```bash
# 1. Initial setup (run once)
chmod +x build-docker.sh compose-build.sh

# 2. For Docker development
./build-docker.sh
docker run -p 8080:8080 guyhaliva/myapp:latest

# 3. For Docker Compose development
./compose-build.sh

# 4. For Kubernetes testing (after building image)
./build-docker.sh
helm install myapp-dev helm/myapp
```

#### **Troubleshooting Scripts:**

```bash
# Check if scripts are executable
ls -la *.sh

# If not executable, fix permissions
chmod +x *.sh

# Test version detection
cd myapp && mvn help:evaluate -Dexpression=project.version -q -DforceStdout && cd ..

# Test Docker build manually
cd myapp
VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
mvn clean package
docker build --build-arg JAR_VERSION=$VERSION -t test-myapp .
cd ..
```

### Kubernetes Deployment with Helm:

#### Prerequisites Setup:

```bash
# Enable Kubernetes in Docker Desktop (GUI):
# Docker Desktop → Settings → Kubernetes → Enable Kubernetes → Apply & Restart

# Verify Kubernetes is running
kubectl cluster-info
kubectl get nodes

# Verify Helm is installed
helm version
```

#### Deploy Application:

```bash
# Navigate to project root
cd /path/to/your/project

# Validate Helm chart
helm lint helm/myapp

# Dry run to preview deployment
helm install myapp-dev helm/myapp --dry-run --debug

# Deploy the application
helm install myapp-dev helm/myapp

# Verify deployment
kubectl get pods # copy the NAME of the pod
kubectl logs POD_NAME -f # replace the POD_NAME with the name you just copied to see the logs from the container
kubectl get services
kubectl get deployments
```

#### Test the Deployed Application:

```bash
# Check pod status and logs
kubectl get pods -l app.kubernetes.io/name=maven-app
kubectl logs -l app.kubernetes.io/name=myapp -f

# Port forward to access the application locally
kubectl port-forward service/myapp-dev 8080:8080

# In another terminal, test the application
curl http://localhost:8080
# Or open http://localhost:8080 in your browser
```

#### Monitor and Debug:

```bash
# Describe pods for detailed information
kubectl describe pods -l app.kubernetes.io/name=myapp

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Watch pod status in real-time
kubectl get pods -l app.kubernetes.io/name=myapp -w

# Get all resources created by the Helm chart
kubectl get all -l app.kubernetes.io/name=myapp
```

#### Management Commands:

```bash
# List Helm releases
helm list

# Upgrade deployment (after code changes)
helm upgrade myapp-dev helm/myapp

# Rollback to previous version
helm rollback myapp-dev

# Uninstall the application
helm uninstall myapp-dev

# Restart deployment (useful after rebuilding Docker image)
kubectl rollout restart deployment/myapp-dev

# Scale the application
kubectl scale deployment/myapp-dev --replicas=3
```

## Java Version Compatibility

- **Minimum Required**: JDK 13
- **Pipeline Uses**: JDK 17 (LTS version)
- **Docker Runtime**: Eclipse Temurin 17
- **Compiler Target**: Java 17 bytecode

## JVM Optimizations

The Docker container includes modern JVM flags:

- `UseContainerSupport`: Automatically detects container limits
- `MaxRAMPercentage=75.0`: Uses 75% of available container memory
- Optimized for containerized environments

## Pipeline Triggers

- **Push to main**: Full pipeline (test → build → docker → deploy)
- **Push to develop**: Test and build only
- **Pull Request**: Test only

## Monitoring

- Check GitHub Actions tab for pipeline status
- Docker images available at: `https://hub.docker.com/r/YOUR_USERNAME/myapp`
- Artifacts stored for 30 days in GitHub

## Troubleshooting

1. **Java Version Issues**: Ensure local development uses JDK 13+
2. **Docker Login Issues**: Verify DOCKER_USERNAME and DOCKER_PASSWORD secrets
3. **Maven Issues**: Check pom.xml syntax and dependencies
4. **Container Memory**: Adjust MaxRAMPercentage if needed
5. **Permission Issues**: Docker runs as non-root user for security
6. **Kubernetes Issues**:
   - **Cluster not running**: Verify Docker Desktop Kubernetes is enabled and running
   - **Pod failing to start**: `kubectl describe pod POD_NAME` to see events and error messages
   - **Image pull issues**: Ensure Docker image exists locally (use `docker images | grep myapp`)
   - **Service not accessible**: Check service endpoints with `kubectl get endpoints`
7. **Helm Issues**:
   - **Chart validation**: Run `helm lint helm/myapp` to check for syntax errors
   - **Release conflicts**: Use `helm list` to check existing releases
   - **Failed deployment**: Use `helm get all RELEASE_NAME` for detailed debugging
   - **Values issues**: Verify `helm/myapp/values.yaml` image repository matches your Docker Hub username
8. **Application Logging**:
   - **No logs appearing**: Check if application is writing to stdout/stderr
   - **Pod crashes**: Use `kubectl logs POD_NAME --previous` to see logs from crashed containers
   - **Log aggregation**: Use `kubectl logs -l app.kubernetes.io/name=myapp --tail=100 -f` for real-time logs

## Modern Java Features Available

With JDK 17, you can use:

- Text blocks (JDK 15+)
- Pattern matching for instanceof (JDK 16+)
- Sealed classes (JDK 17)
- Records (JDK 14+)
- Switch expressions (JDK 14+)

## GitHub Actions Workflow

To set up the GitHub Actions workflow, create the file `.github/workflows/ci-cd.yml` with the pipeline configuration. The workflow includes:

- **Test Job**: Runs unit tests with JDK 17
- **Build Job**: Compiles, packages, and uploads artifacts
- **Docker Job**: Builds and pushes Docker images
- **Deploy Job**: Deployment notifications

## Quick Start

1. **Clone/Fork this repository**
2. **Set up Docker Hub secrets** in GitHub repository settings
3. **Push to main branch** to trigger the full pipeline
4. **Monitor progress** in GitHub Actions tab
5. **Check Docker Hub** for published images

## Example Commands

```bash
# Local build and test
mvn clean test package

# Docker build with proper versioning
docker build --build-arg JAR_VERSION=1.0.0 -t guyhaliva/myapp:1.0.0 .
docker tag guyhaliva/myapp:1.0.0 guyhaliva/myapp:latest

# Docker run and test
docker run --rm -p 8080:8080 guyhaliva/myapp:1.0.0

# Docker Compose
docker-compose up --build

# Kubernetes deployment workflow
helm lint helm/myapp                                    # Validate chart
helm install myapp-dev helm/myapp --dry-run --debug    # Preview deployment
helm install myapp-dev helm/myapp                      # Deploy application
kubectl get all -l app.kubernetes.io/name=myapp        # Check all resources
kubectl logs -l app.kubernetes.io/name=myapp -f        # View logs
kubectl port-forward service/myapp-dev 8080:8080       # Access locally

# Management and updates
helm upgrade myapp-dev helm/myapp                       # Update deployment
kubectl rollout restart deployment/myapp-dev           # Restart after image update
kubectl scale deployment/myapp-dev --replicas=3        # Scale application
helm uninstall myapp-dev                               # Remove deployment

# Debugging commands
kubectl describe pods -l app.kubernetes.io/name=myapp  # Detailed pod info
kubectl get events --sort-by=.metadata.creationTimestamp # Recent events
helm get all myapp-dev                                 # Full Helm release info

# Check application output
curl http://localhost:8080  # if you add a web server
```
