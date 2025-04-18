pipeline {
    agent any
    
    environment {
        // Simplified environment for local usage
        IMAGE_NAME = 'atomic-crm'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG_ID = 'kubeconfig' // Jenkins credential ID for Kubernetes config
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
                    withCredentials([file(credentialsId: "${KUBE_CONFIG_ID}", variable: 'KUBECONFIG')]) {
                        // Use $KUBECONFIG directly for each command
                        powershell "kubectl --kubeconfig=\"${env:KUBECONFIG}\" apply -f k8s-deploy.yaml"
                        powershell "kubectl --kubeconfig=\"${env:KUBECONFIG}\" set image deployment/atomic-crm atomic-crm=atomic-crm:${IMAGE_TAG}"
                        powershell "kubectl --kubeconfig=\"${env:KUBECONFIG}\" rollout status deployment/atomic-crm"
                    }
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
