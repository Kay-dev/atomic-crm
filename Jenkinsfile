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
                    // powershell '''
                    //   echo "Deploying to Kubernetes...";
                    // '''
                    withCredentials([file(credentialsId: "${KUBE_CONFIG_ID}", variable: 'KUBECONFIG')]) {
                        powershell '''
                            $kubeconfigPath = $env:KUBECONFIG
                            kubectl --kubeconfig="$kubeconfigPath" apply -f k8s-deploy.yaml --validate=false
                            
                            # Add a short delay to ensure the deployment is registered
                            Start-Sleep -Seconds 5
                            
                            # Use -n default to explicitly specify the namespace
                            kubectl --kubeconfig="$kubeconfigPath" get deployments -n default
                            kubectl --kubeconfig="$kubeconfigPath" set image deployment/atomic-crm atomic-crm=atomic-crm:${env:IMAGE_TAG} -n default
                            kubectl --kubeconfig="$kubeconfigPath" rollout status deployment/atomic-crm -n default
                        '''
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
