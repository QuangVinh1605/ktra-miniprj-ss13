# ─────────────────────────────────────────────────────────────────────────────
# Campus Hub Backend — Dockerfile
#
# Build strategy (two steps):
#
#   Step 1 — build JAR on the host (uses local Gradle cache, no network):
#     cd backend && ./gradlew bootJar -x test
#
#   Step 2 — package into Docker image:
#     docker build -t campus-hub-backend:latest ./backend
#
# The final image contains only the JRE and the application JAR (~60 MB).
# Build tools (JDK, Gradle) are NOT included in the runtime image.
# ─────────────────────────────────────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine

# Non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy the fat JAR produced by ./gradlew bootJar
COPY build/libs/*.jar app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
