pipeline {

    agent any

    environment {
        COMPOSE_PROJECT_NAME = "smartclassroom"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                '''
            }
        }

        stage('Validate Compose') {
            steps {
                sh '''
                    docker compose config
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker compose build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose up -d --remove-orphans
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                sh '''
                    echo "Waiting for SmartClassroom services..."
                    sleep 20
                    docker compose ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "Testing SmartClassroom frontend..."
                    curl --fail http://host.docker.internal/ || exit 1

                    echo "Testing FastAPI..."
                    curl --fail http://host.docker.internal/docs || exit 1

                    echo "SmartClassroom is healthy!"
                '''
            }
        }
    }

    post {

        success {
            echo '''
======================================
 SmartClassroom Deployment SUCCESSFUL
======================================
'''
        }

        failure {
            echo '''
======================================
 SmartClassroom Deployment FAILED
======================================
'''

            sh '''
                docker compose ps || true
                docker compose logs --tail=100 || true
            '''
        }

        always {
            echo 'Jenkins pipeline completed.'
        }
    }
}