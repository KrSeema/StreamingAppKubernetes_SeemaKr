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

                    echo "===== Building Auth Image ====="
                    docker build \
                        -f backend/authService/Dockerfile \
                        -t streaming-auth:v1 \
                        ./backend

                    echo "===== Building Streaming Image ====="
                    docker build \
                        -f backend/streamingService/Dockerfile \
                        -t streaming-stream:v1 \
                        ./backend

                    echo "===== Building Admin Image ====="
                    docker build \
                        -f backend/adminService/Dockerfile \
                        -t streaming-admin:v1 \
                        ./backend

                    echo "===== Building Chat Image ====="
                    docker build \
                        -f backend/chatService/Dockerfile \
                        -t streaming-chat:v1 \
                        ./backend

                    echo "===== Building Frontend Image ====="
                    docker build \
                        -f frontend/Dockerfile \
                        -t streaming-frontend:v1 \
                        ./frontend

                    echo "===== Docker Images ====="
                    docker images --format "{{.Repository}}:{{.Tag}}" | grep "streaming-"
                '''
            }
        }
    }
}