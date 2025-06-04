# use Eclipse Temurin JRE 17 as base image (supports JDK 13+)
FROM eclipse-temurin:17-jre

# accept version as build argument
ARG JAR_VERSION
ENV JAR_VERSION=${JAR_VERSION}

# set working directory
WORKDIR /app

# copy the specific JAR file using the version argument
COPY target/myapp-${JAR_VERSION}.jar app.jar

# create a non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app
USER appuser

# expose port (optional, for documentation)
EXPOSE 8080

# health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD java -version || exit 1

# run the application with JVM optimizations for containerized environments
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"] 