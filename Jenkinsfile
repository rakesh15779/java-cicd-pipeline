pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rakesh15779/java-cicd-pipeline.git'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'java -version'
                sh 'mvn -B clean test package'
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
                    docker run -d --name java-cicd-app \
                        --restart unless-stopped \
                        -p 8081:8080 \
                        java-cicd-pipeline:latest
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                    sleep 5
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

