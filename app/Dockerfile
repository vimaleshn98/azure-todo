FROM maven:3.5-jdk-8
COPY /var/lib/jenkins/workspace/TodoApp@2/app/target/*.war app.war
ENTRYPOINT [ "java","-jar","app.war" ]
