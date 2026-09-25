pipeline {

    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'ecommerce'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    set -eu

                    echo "Current commit:"
                    git rev-parse --short HEAD

                    test -f backend/Dockerfile
                    test -f frontend/Dockerfile
                    test -f frontend/nginx.conf
                    test -f docker-compose.yml

                    echo "Required files exist."
                '''
            }
        }

        stage('Docker Check') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                    docker info >/dev/null
                '''
            }
        }

        stage('Compose Validate') {
            steps {
                sh '''
                    docker compose config -q
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker compose build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose up -d
                '''
            }
        }

        stage('Status') {
            steps {
                sh '''
                    docker compose ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10

                    curl --fail \
                         --retry 10 \
                         --retry-delay 3 \
                         --retry-connrefused \
                         http://localhost/

                    echo "Website is responding."
                '''
            }
        }
    }

    post {

        success {
            echo 'Hurray E-commerce application deployed successfully.'
        }

        failure {
            sh '''
                docker compose ps || true

                docker compose logs --tail=100 backend || true

                docker compose logs --tail=100 db || true

                docker compose logs --tail=100 frontend || true
            '''
        }
    }
}
