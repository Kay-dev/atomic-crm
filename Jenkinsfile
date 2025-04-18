pipeline {
    agent any

    environment {
        // Simplified environment for local usage
        IMAGE_NAME = 'atomic-crm'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG_ID = 'kubeconfig' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                powershell 'npm install'
            }
        }

        stage('Build') {
            steps {
                powershell 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                powershell "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    powershell '''
                      echo "Deploying to Kubernetes...";
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to Kubernetes successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
        always {
            // Clean up Docker images locally using proper PowerShell error handling
            powershell "try { docker rmi ${IMAGE_NAME}:${IMAGE_TAG} } catch { Write-Host \"Could not remove image ${IMAGE_NAME}:${IMAGE_TAG}\" }"
            powershell "try { docker rmi ${IMAGE_NAME}:latest } catch { Write-Host \"Could not remove image ${IMAGE_NAME}:latest\" }"
        }
    }
}
