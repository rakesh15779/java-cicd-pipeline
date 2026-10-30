pipeline {
    agent any

    environment {
        UBUNTU_HOST = '172.26.17.92'
        APP_NAME = 'java-cicd-app'
        IMAGE_NAME = 'java-cicd-pipeline:latest'
        REMOTE_DIR = '/opt/java-cicd-pipeline'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub'
            }
        }

        stage('Build and Test') {
            steps {
                bat 'mvn clean test package'
            }
        }

        stage('Prepare SSH Key') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    powershell '''
                        $sourceKey = $env:SSH_KEY
                        $secureKey = Join-Path $env:WORKSPACE "jenkins_ubuntu_key"

                        Copy-Item -LiteralPath $sourceKey -Destination $secureKey -Force

                        icacls $secureKey /inheritance:r | Out-Null
                        icacls $secureKey /grant:r "SYSTEM:F" | Out-Null

                        if ($LASTEXITCODE -ne 0) {
                            throw "Failed to set SSH key permissions"
                        }

                        "SSH_KEY_PATH=$secureKey" |
                            Out-File -FilePath "$env:WORKSPACE\\ssh-path.txt" -Encoding ascii
                    '''
                }
            }
        }

        stage('Copy JAR to Ubuntu') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    powershell '''
                        $secureKey = "$env:WORKSPACE\\jenkins_ubuntu_key"

                        icacls $secureKey /inheritance:r | Out-Null
                        icacls $secureKey /grant:r "SYSTEM:F" | Out-Null

                        scp -i $secureKey -o StrictHostKeyChecking=no `
                            "$env:WORKSPACE\\target\\java-cicd-pipeline-1.0.0.jar" `
                            "$env:SSH_USER@$env:UBUNTU_HOST:$env:REMOTE_DIR/target/java-cicd-pipeline-1.0.0.jar"

                        if ($LASTEXITCODE -ne 0) {
                            throw "JAR copy to Ubuntu failed"
                        }
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    powershell '''
                        $secureKey = "$env:WORKSPACE\\jenkins_ubuntu_key"

                        ssh -i $secureKey -o StrictHostKeyChecking=no `
                            "$env:SSH_USER@$env:UBUNTU_HOST" `
                            "docker build -t java-cicd-pipeline:latest /opt/java-cicd-pipeline"

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker image build failed"
                        }
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
                    powershell '''
                        $secureKey = "$env:WORKSPACE\\jenkins_ubuntu_key"

                        ssh -i $secureKey -o StrictHostKeyChecking=no `
                            "$env:SSH_USER@$env:UBUNTU_HOST" `
                            "docker rm -f java-cicd-app 2>/dev/null || true; docker run -d --name java-cicd-app -p 8081:8080 java-cicd-pipeline:latest"

                        if ($LASTEXITCODE -ne 0) {
                            throw "Container deployment failed"
                        }
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
                    powershell '''
                        $secureKey = "$env:WORKSPACE\\jenkins_ubuntu_key"

                        ssh -i $secureKey -o StrictHostKeyChecking=no `
                            "$env:SSH_USER@$env:UBUNTU_HOST" `
                            "curl --fail http://localhost:8081/ && curl --fail http://localhost:8081/health"

                        if ($LASTEXITCODE -ne 0) {
                            throw "Application health verification failed"
                        }
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

        always {
            powershell '''
                Remove-Item "$env:WORKSPACE\\jenkins_ubuntu_key" -Force -ErrorAction SilentlyContinue
                Remove-Item "$env:WORKSPACE\\ssh-path.txt" -Force -ErrorAction SilentlyContinue
            '''
        }
    }
}

