# ---------- build stage ----------
FROM maven:3.9.4-eclipse-temurin-17 AS build
WORKDIR /app

# copy only what we need to cache dependencies smartly
COPY pom.xml .
COPY settings.xml .     # optional: if you use a custom maven settings.xml
RUN mvn -B -DskipTests dependency:go-offline

COPY src ./src
RUN mvn -B -DskipTests clean package

# ---------- runtime stage ----------
FROM eclipse-temurin:17-jre
WORKDIR /app

# copy jar produced by the build stage (assumes single jar in target/)
COPY --from=build /app/target/*.jar app.jar

# runtime tuning
ENV JAVA_OPTS="-Xms256m -Xmx512m" \
    PORT=8080

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- --tries=1 --timeout=2 http://127.0.0.1:${PORT}/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Dserver.port=${PORT} -jar /app/app.jar"]
