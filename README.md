# Java Maven CI/CD Pipeline

## Project Overview

This project demonstrates an automated CI/CD pipeline for building, testing, and deploying a Java Spring Boot application using Jenkins, Maven, Docker, Git, and GitHub.

## Technologies Used

- Linux (Ubuntu)
- Java 21
- Git and GitHub
- Maven
- Jenkins
- Docker
- Spring Boot

## CI/CD Workflow

1. Fetch source code from GitHub.
2. Build and test the application using Maven.
3. Build a Docker image.
4. Deploy the application in a Docker container.
5. Verify application health using HTTP endpoints.

## Project Structure

```text
java-cicd-pipeline/
├── src/
├── target/
├── Dockerfile
├── Jenkinsfile
├── pom.xml
└── README.md
```

## Application URLs

- Application: `http://localhost:8081/`
- Health Check: `http://localhost:8081/health`

## Project Status

**Successfully Completed**

- Jenkins CI/CD pipeline configured.
- Maven build and unit tests passed.
- Docker image created successfully.
- Application deployed in a Docker container.
- Application health verified.

## How to Run

```bash
git clone https://github.com/rakesh15779/java-cicd-pipeline.git
cd java-cicd-pipeline
mvn clean test package
docker build -t java-cicd-pipeline:latest .
docker run -d --name java-cicd-app -p 8081:8080 java-cicd-pipeline:latest
```

## Author

Rakesh Kumar

GitHub: https://github.com/rakesh15779
