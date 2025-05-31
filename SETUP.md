# CI/CD Pipeline Setup Instructions

## Prerequisites

1. **GitHub Repository**: Ensure your code is in a GitHub repository
2. **Docker Hub Account**: Create an account at https://hub.docker.com
3. **Java 17**: Application requires JDK 13+ (pipeline uses JDK 17)
4. **GitHub Secrets**: Configure the following secrets in your GitHub repository

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

### Build and Run Locally:

```bash
# Ensure you have JDK 13+ installed
java -version

# Build the project
cd myapp
mvn clean package

# Run the JAR
java -jar target/myapp-*.jar

# Or use Docker Compose
docker-compose up --build
```

### Test the Pipeline Locally:

```bash
# Test compilation
cd myapp
mvn clean compile

# Test packaging
mvn clean package

# Test Docker build
docker build -t myapp:test .

# Run Docker container
docker run --rm myapp:test
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
- Docker images available at: `https://hub.docker.com/r/guyhaliva/myapp`
- Artifacts stored for 30 days in GitHub

## Troubleshooting

1. **Java Version Issues**: Ensure local development uses JDK 13+
2. **Docker Login Issues**: Verify DOCKER_USERNAME and DOCKER_PASSWORD secrets
3. **Maven Issues**: Check pom.xml syntax and dependencies
4. **Container Memory**: Adjust MaxRAMPercentage if needed
5. **Permission Issues**: Docker runs as non-root user for security

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

# Docker build and run
docker build -t myapp .
docker run --rm myapp

# Docker Compose
docker-compose up --build

# Check application output
curl http://localhost:8080  # if you add a web server
```

## Next Steps

- Add web endpoints to make it a REST API
- Add database connectivity
- Implement health checks
- Add monitoring and logging
- Set up staging/production environments
