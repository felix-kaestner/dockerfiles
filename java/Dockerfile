ARG JAVA_VERSION=17
ARG GRADLE_VERSION=7.4

FROM gradle:${GRADLE_VERSION}-jdk${JAVA_VERSION} as build

# Switch into workspace
WORKDIR /workspace

# Copy the files into the container's workspace.
COPY build.gradle.kts settings.gradle.kts gradle.properties /workspace/
COPY src /workspace/src

# Build application
RUN gradle assemble --no-daemon

FROM eclipse-temurin:${JAVA_VERSION} as jre-build

ARG JAVA_VERSION

# Copy jar-application from build
COPY --from=build /workspace/build/libs/*-fat.jar /app/server.jar

# Find JDK dependencies dynamically from jar
RUN jdeps \
    -q \
    --multi-release ${JAVA_VERSION} \
    --print-module-deps \
    --ignore-missing-deps \
    /app/server.jar > jre-deps.info

# Create a custom Java runtime
RUN jlink \
    --add-modules $(cat jre-deps.info) \
    --strip-debug \
    --no-man-pages \
    --no-header-files \
    --compress=2 \
    --output /javaruntime

FROM gcr.io/distroless/java-base-debian11:nonroot as jre

ENV JAVA_HOME=/opt/java/openjdk
ENV PATH "${JAVA_HOME}/bin:${PATH}"

# Copy custom java runtime environment
COPY --from=jre-build /javaruntime $JAVA_HOME

FROM jre

# Switch into workspace
WORKDIR /app

# Copy jar-application from build
COPY --from=build --chown=nonroot:nonroot /workspace/build/libs/*-fat.jar /app/server.jar

# Execute the server application
CMD ["java", "-jar", "/app/server.jar"]
