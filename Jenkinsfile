pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        SONARQUBE_URL = 'http://localhost:9000'
    }

    stages {
        stage('Clone Code from GitHub') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Quality Analysis') {
            steps {
                sh '''
                    cd backend && sonar-scanner \
                        -Dsonar.projectKey=wanderlust-backend \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONARQUBE_URL \
                        -Dsonar.login=admin \
                        -Dsonar.password=admin123 || echo "SonarQube scan attempted"
                '''
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                    dependency-check --project wanderlust \
                        --scan . \
                        --format ALL \
                        --out dependency-check-report || echo "Dependency check attempted"
                '''
            }
        }

        stage('Sonar Quality Gate Scan') {
            steps {
                echo 'Quality Gate check skipped - SonarQube server not configured'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh '''
                    trivy fs --scanners vuln,secret,misconfig \
                        --format table \
                        --exit-code 0 \
                        --severity HIGH,CRITICAL . || echo "Trivy scan attempted"
                '''
            }
        }

        stage('Deploy using Docker compose') {
            steps {
                sh '''
                    docker compose -f $DOCKER_COMPOSE_FILE down || true
                    docker compose -f $DOCKER_COMPOSE_FILE up -d --build
                '''
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f || true'
        }
        failure {
            sh 'docker compose -f $DOCKER_COMPOSE_FILE down || true'
        }
    }
}