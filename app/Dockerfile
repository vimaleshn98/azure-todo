FROM maven:3.5-jdk-8
COPY app/target/*.war app.war
ENTRYPOINT [ "java","-jar","app.war" ]
