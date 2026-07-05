pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {
        stage('Clone Code from GitHub') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        cd backend && sonar-scanner \
                            -Dsonar.projectKey=wanderlust-backend \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=$SONARQUBE_URL \
                            -Dsonar.login=$SONARQUBE_TOKEN
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '''
                    --project wanderlust \
                    --scan . \
                    --format ALL \
                    --out dependency-check-report
                ''', odcInstallation: 'OWASP-Dependency-Check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

        stage('Sonar Quality Gate Scan') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh '''
                    trivy fs --scanners vuln,secret,misconfig \
                        --format table \
                        --exit-code 0 \
                        --severity HIGH,CRITICAL .
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