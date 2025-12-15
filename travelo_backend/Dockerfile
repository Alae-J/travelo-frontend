# ==============================
# STAGE 1: Build the JAR (if needed)
# ==============================

FROM eclipse-temurin:23-jdk-alpine AS builder
WORKDIR /app

# Copy Maven wrapper explicitly
COPY .mvn/ .mvn/
COPY mvnw .
RUN chmod +x mvnw

# Copy pom.xml and build dependencies
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B

# Copy source and build JAR
COPY src ./src
RUN ./mvnw package -DskipTests

# ==============================
# STAGE 2: Runtime (Minimal & Secure)
# ==============================
FROM eclipse-temurin:23-jre-alpine
RUN addgroup -g 1001 -S appgroup && adduser -u 1001 -S appuser -G appgroup
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN chown appuser:appgroup app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]