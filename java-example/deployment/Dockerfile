FROM openjdk:8-jre-alpine

COPY *.war /lib/app.war

RUN echo "Asia/Shanghai" > /etc/timezone

CMD ["java", "-jar", "/lib/app.war"]
