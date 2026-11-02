pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean test package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t java-cicd-pipeline:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f java-cicd-app || true
                    docker run -d \
                        --name java-cicd-app \
                        -p 8081:8080 \
                        java-cicd-pipeline:latest
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                    sleep 10
                    curl --fail http://localhost:8081/
                    curl --fail http://localhost:8081/health
                '''
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
