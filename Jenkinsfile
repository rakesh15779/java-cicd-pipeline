
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
                sh 'java -version'
                sh '/usr/share/maven/bin/mvn -B clean test package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t java-cicd-pipeline:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker rm -f java-cicd-app || true'
                sh 'docker run -d --name java-cicd-app -p 8081:8080 java-cicd-pipeline:latest'
            }
        }

        stage('Verify Application') {
            steps {
                sh 'curl --fail http://localhost:8081/'
                sh 'curl --fail http://localhost:8081/health'
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

