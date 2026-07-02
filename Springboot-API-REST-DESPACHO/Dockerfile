# ══ STAGE 1: Build con Maven ═════════════════════════
FROM maven:3.9-eclipse-temurin-17-alpine AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -B \
    -Dmaven.wagon.http.ssl.insecure=true \
    -Dmaven.wagon.http.ssl.allowall=true \
    -Dhttp.userAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
    
COPY src ./src
RUN mvn clean package -DskipTests -B

# ══ STAGE 2: Imagen de producción ════════════════════
FROM eclipse-temurin:17-jre-alpine

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

RUN chown appuser:appgroup app.jar

USER appuser

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]