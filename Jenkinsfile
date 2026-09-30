
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Order API Image') {
            steps {
                bat 'docker compose build order-api'
            }
        }

        stage('Start Application') {
            steps {
                bat 'docker compose up -d'
            }
        }

        stage('Wait for Services') {
            steps {
                bat '''
                powershell -Command "$max=60; for($i=1; $i -le $max; $i++){ try { $r=Invoke-WebRequest -Uri http://localhost:8095/health -UseBasicParsing; if($r.StatusCode -eq 200){ Write-Host 'Application is ready'; exit 0 } } catch {} ; Write-Host 'Waiting for application...'; Start-Sleep -Seconds 2 }; Write-Error 'Application did not become ready'; exit 1"
                '''
            }
        }

        stage('Health Check') {
            steps {
                bat 'curl -f http://localhost:8095/health'
            }
        }

        stage('Create Test Order') {
            steps {
                bat '''
                curl -X POST http://localhost:8095/orders ^
                -H "Content-Type: application/json" ^
                -d "{\"customer_name\":\"Jenkins\",\"food_item\":\"Burger\",\"quantity\":2}"
                '''
            }
        }

        stage('Verify Order') {
            steps {
                bat 'curl -f http://localhost:8095/orders'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Displaying container logs...'
            bat 'docker compose logs --no-color'
        }

        always {
            echo 'Stopping application containers...'
            bat 'docker compose down'
        }
    }
}

