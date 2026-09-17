pipeline {

    agent any

    environment {
        DOCKERHUB_USERNAME = 'kitkat1128'

        BACKEND_IMAGE = "${DOCKERHUB_USERNAME}/air-quality-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USERNAME}/air-quality-frontend"

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                '''
            }
        }

        stage('Build Backend') {
            steps {
                sh '''
                    docker build \
                      -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                      ./backend
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    docker build \
                      --build-arg VITE_API_URL=${VITE_API_URL} \
                      -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                      ./frontend
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'Dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Backend') {
            steps {
                sh '''
                    docker push \
                    ${BACKEND_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Frontend') {
            steps {
                sh '''
                    docker push \
                    ${FRONTEND_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Stop Existing Containers') {
            steps {
                sh '''
                    echo "Stopping existing application containers..."

                    docker compose \
                    -f docker-compose.yml \
                    down || true
                '''
            }
        }

        stage('Deploy Production') {
            steps {
                sh '''
                    echo "Deploying production containers..."

                    export IMAGE_TAG=${IMAGE_TAG}
                    export DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME}

                    docker compose \
                    -f docker-compose.prod.yml \
                    pull

                    docker compose \
                    -f docker-compose.prod.yml \
                    up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Checking running containers..."

                    docker compose \
                    -f docker-compose.prod.yml \
                    ps
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
