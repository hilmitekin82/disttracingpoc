FROM hilmit82/jdk11-maven-builder:alpine3.9 AS builder
WORKDIR /app
COPY service/ /app
RUN mvn package
RUN wget -O apm-agent.jar https://search.maven.org/remotecontent?filepath=co/elastic/apm/elastic-apm-agent/1.6.1/elastic-apm-agent-1.6.1.jar
RUN jar -xvf target/apm-0.0.1-SNAPSHOT.jar && cd BOOT-INF/lib \
    && for FILE in `find`;do jdeps --list-deps $FILE | grep -E 'java.|jdk.' | tee -a java.output;done \
    && tr -d [:blank:] < java.output | tr "\n" "," > java.output.comma \
    && cat  java.output.comma \
    && jlink \
        --verbose \
        --module-path "$JAVA_HOME/jmods" \
        --add-modules `cat  java.output.comma` \
        --compress 2 \
        --no-header-files \
        --no-man-pages \
        --strip-debug \
        --output /opt/jre

FROM hilmit82/alpine-glibc:3.9
WORKDIR /usr/src/app
ENV JAVA_HOME=/opt/jre
ENV LANG=C.UTF-8
ENV PATH="$PATH:$JAVA_HOME/bin"
ENV APP_TIMEZONE=UTC

COPY --from=builder /opt/jre /opt/jre
COPY --from=builder /app/target/*.jar app.jar
COPY --from=builder /app/apm-agent.jar apm-agent.jar
EXPOSE 8080
CMD ["java","-javaagent:apm-agent.jar","-jar","app.jar"]