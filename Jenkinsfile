pipeline {
    agent any

    environment {
        UBUNTU_HOST = '172.26.17.92'
        SSH_CREDENTIALS = 'ubuntu-ssh'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Build and Test') {
            steps {
                bat 'mvn clean test package'
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat '''
                    scp -i "%SSH_KEY%" -o StrictHostKeyChecking=no target\\java-cicd-pipeline-1.0.0.jar %SSH_USER%@%UBUNTU_HOST%:/opt/java-cicd-pipeline/target/java-cicd-pipeline-1.0.0.jar
                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%UBUNTU_HOST% "docker build -t java-cicd-pipeline:latest /opt/java-cicd-pipeline"
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat '''
                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%UBUNTU_HOST% "docker rm -f java-cicd-app || true"
                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%UBUNTU_HOST% "docker run -d --name java-cicd-app -p 8081:8080 java-cicd-pipeline:latest"
                    '''
                }
            }
        }

        stage('Verify Application') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat '''
                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%UBUNTU_HOST% "curl --fail http://localhost:8081/"
                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %SSH_USER%@%UBUNTU_HOST% "curl --fail http://localhost:8081/health"
                    '''
                }
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
