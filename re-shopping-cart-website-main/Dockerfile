# Multi-stage build for production-ready Java 21 Docker image

# Stage 1: Builder
FROM gradle:8.5-jdk21 as builder

WORKDIR /app

# Copy project files
COPY build.gradle settings.gradle ./
COPY gradle ./gradle
COPY src ./src

# Build the application
RUN gradle clean build -x test --no-daemon

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine

LABEL maintainer="DevOps Team"
LABEL description="RE Shopping Cart Web App - Production Docker Image"

# Create non-root user for security
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /app

# Copy JAR from builder stage
COPY --from=builder /app/build/libs/*.jar app.jar

# Ensure correct permissions
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

# Expose port
EXPOSE 8080

# Set JVM options for production
ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ParallelRefProcEnabled -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9010 -Dcom.sun.management.jmxremote.rmi.port=9010 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"

# Start application
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -Dspring.profiles.active=${SPRING_PROFILES_ACTIVE:-prod} -jar app.jar"]
