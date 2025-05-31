# Use Eclipse Temurin JRE 17 as base image (supports JDK 13+)
FROM eclipse-temurin:17-jre

# Accept version as build argument
ARG JAR_VERSION
ENV JAR_VERSION=${JAR_VERSION}

# Set working directory
WORKDIR /app

# Copy the specific JAR file using the version argument
COPY target/myapp-${JAR_VERSION}.jar app.jar

# Create a non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app
USER appuser

# Expose port (optional, for documentation)
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD java -version || exit 1

# Run the application with JVM optimizations for containerized environments
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"] 