
pipeline {
    agent any

    environment {
        DB_NAME = 'ordersdb'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                bat 'docker --version'
                bat 'docker-compose version'
            }
        }

        stage('Build Order API Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'food-db-credentials',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    bat 'docker-compose build order-api'
                }
            }
        }

        stage('Start Application') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'food-db-credentials',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    bat 'docker-compose up -d'
                }
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
                -d "{\\"customer_name\\":\\"Jenkins\\",\\"food_item\\":\\"Burger\\",\\"quantity\\":2}"
                '''
            }
        }

        stage('Verify Order') {
            steps {
                bat 'curl -f http://localhost:8095/orders'
            }
        }

        stage('Verify Database') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'food-db-credentials',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    bat '''
                    docker exec food-db psql -U %DB_USER% -d %DB_NAME% -c "SELECT id, customer_name, food_item, quantity FROM orders ORDER BY id;"
                    '''
                }
            }
        }
    }

    post {

        failure {
            echo 'Pipeline failed. Displaying container logs...'

            withCredentials([
                usernamePassword(
                    credentialsId: 'food-db-credentials',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PASSWORD'
                )
            ]) {
                bat 'docker-compose logs'
            }
        }

        always {
            echo 'Stopping application containers...'

            withCredentials([
                usernamePassword(
                    credentialsId: 'food-db-credentials',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PASSWORD'
                )
            ]) {
                bat 'docker-compose down'
            }
        }
    }
}

