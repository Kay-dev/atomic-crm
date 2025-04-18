pipeline {
    agent any
    
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
        
        stage('Start Application') {
            steps {
                // Stop any running instance first
                powershell 'powershell -Command "Get-Process -Name node -ErrorAction SilentlyContinue | Where-Object {$_.CommandLine -like \"*atomic-crm*\"} | Stop-Process -Force -ErrorAction SilentlyContinue"'
                
                // Start the application (non-blocking)
                powershell '''
                    Start-Process -FilePath "npm" -ArgumentList "run", "serve", "--", "--port", "3000" -NoNewWindow
                    Write-Host "Application started on http://localhost:3000"
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
