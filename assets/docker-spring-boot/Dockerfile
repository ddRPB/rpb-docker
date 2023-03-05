# For Java 8
# FROM openjdk:8-jdk-alpine
# For Java 11, try this
#FROM adoptopenjdk/openjdk11:alpine-jre
FROM adoptopenjdk/openjdk11:alpine-jre

ARG JAR_FILE=target/spring-boot-web.jar

WORKDIR /opt/app

# cp target/spring-boot-web.jar /opt/app/app.jar
COPY ${JAR_FILE} app.jar

# java -jar /opt/app/app.jar
ENTRYPOINT ["java","-jar","app.jar"]
