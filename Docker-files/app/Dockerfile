# syntax=docker/dockerfile:1
# VProfile - App Image (Tomcat + vprofile artifact)
# Multi-stage build to keep the final image small.

FROM maven:3.9.9-eclipse-temurin-21-jammy AS build_image

# Build the artifact
RUN git clone https://github.com/hkhcoder/vprofile-project.git \
    && cd vprofile-project \
    && git checkout containers \
    && mvn -q -DskipTests clean install

FROM tomcat:10-jdk21

LABEL Project="vprofile" \
      Component="app" \
      Maintainer="your-name"

# Remove default apps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy built artifact into Tomcat as ROOT app
COPY --from=build_image /vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh","run"]
