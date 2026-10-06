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
                sh 'docker --version'
                sh 'docker compose version'
            }
        }

        stage('Backend Validation') {
            steps {
                sh '''
                    docker run --rm \
                    -v "$PWD/backend:/app" \
                    -w /app \
                    python:3.13-slim \
                    python -m compileall app api
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose down
                    docker compose up -d
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                sh '''
                    sleep 15
                    docker compose ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    curl --fail http://localhost/ || exit 1
                    curl --fail http://localhost/docs || exit 1
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo ' SmartClassroom Deployment Successful '
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo ' SmartClassroom Deployment Failed '
            echo '======================================'

            sh 'docker compose ps || true'
            sh 'docker compose logs --tail=100 || true'
        }

        always {
            echo 'Jenkins pipeline completed.'
        }
    }
}