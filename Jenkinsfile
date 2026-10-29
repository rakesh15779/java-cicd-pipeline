
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub.'
            }
        }

        stage('Build and Test') {
            steps {
                bat 'java -version'
                bat 'mvn -B clean test package'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t java-cicd-pipeline:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                bat 'docker rm -f java-cicd-app'
                bat 'docker run -d --name java-cicd-app -p 8081:8080 java-cicd-pipeline:latest'
            }
        }

        stage('Verify Application') {
            steps {
                bat 'curl.exe --fail http://localhost:8081/'
                bat 'curl.exe --fail http://localhost:8081/health'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}

