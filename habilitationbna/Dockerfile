# Base image Java 8
FROM openjdk:17-alpine

# Port exposé
EXPOSE 8081

# Copier le JAR généré par Maven
ADD target/habilitationbna-1.0.1.jar  app.jar

# Lancer l’application
ENTRYPOINT ["java", "-jar", "/app.jar"]
