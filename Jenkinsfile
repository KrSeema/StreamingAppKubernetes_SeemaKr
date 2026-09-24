pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    set -e

                    if docker image inspect streaming-auth:${IMAGE_TAG} >/dev/null 2>&1; then
                        echo "Auth image already exists - skipping build"
                    else
                        docker build -f backend/authService/Dockerfile \
                            -t streaming-auth:${IMAGE_TAG} ./backend
                    fi

                    if docker image inspect streaming-stream:${IMAGE_TAG} >/dev/null 2>&1; then
                        echo "Streaming image already exists - skipping build"
                    else
                        docker build -f backend/streamingService/Dockerfile \
                            -t streaming-stream:${IMAGE_TAG} ./backend
                    fi

                    if docker image inspect streaming-admin:${IMAGE_TAG} >/dev/null 2>&1; then
                        echo "Admin image already exists - skipping build"
                    else
                        docker build -f backend/adminService/Dockerfile \
                            -t streaming-admin:${IMAGE_TAG} ./backend
                    fi

                    if docker image inspect streaming-chat:${IMAGE_TAG} >/dev/null 2>&1; then
                        echo "Chat image already exists - skipping build"
                    else
                        docker build -f backend/chatService/Dockerfile \
                            -t streaming-chat:${IMAGE_TAG} ./backend
                    fi

                    if docker image inspect streaming-frontend:${IMAGE_TAG} >/dev/null 2>&1; then
                        echo "Frontend image already exists - skipping build"
                    else
                        docker build -f frontend/Dockerfile \
                            -t streaming-frontend:${IMAGE_TAG} ./frontend
                    fi
                '''
            }
        }
    }
}