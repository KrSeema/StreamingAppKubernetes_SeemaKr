pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '218014315198'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = 'v1'
    }

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
                        -t streaming-auth:${IMAGE_TAG} \
                        ./backend

                    echo "===== Building Streaming Image ====="
                    docker build \
                        -f backend/streamingService/Dockerfile \
                        -t streaming-stream:${IMAGE_TAG} \
                        ./backend

                    echo "===== Building Admin Image ====="
                    docker build \
                        -f backend/adminService/Dockerfile \
                        -t streaming-admin:${IMAGE_TAG} \
                        ./backend

                    echo "===== Building Chat Image ====="
                    docker build \
                        -f backend/chatService/Dockerfile \
                        -t streaming-chat:${IMAGE_TAG} \
                        ./backend

                    echo "===== Building Frontend Image ====="
                    docker build \
                        -f frontend/Dockerfile \
                        -t streaming-frontend:${IMAGE_TAG} \
                        ./frontend
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-cloud',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        set -e

                        echo "===== Logging in to Amazon ECR ====="

                        aws ecr get-login-password \
                            --region ${AWS_REGION} | \
                        docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}
                    '''
                }
                sh '''
                    set -e

                    echo "===== Logging in to Amazon ECR ====="

                    aws ecr get-login-password \
                        --region ${AWS_REGION} | \
                    docker login \
                        --username AWS \
                        --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                    set -e

                    echo "===== Tagging Images for ECR ====="

                    docker tag streaming-auth:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}

                    docker tag streaming-stream:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/streaming-stream:${IMAGE_TAG}

                    docker tag streaming-admin:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}

                    docker tag streaming-chat:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}

                    docker tag streaming-frontend:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    set -e

                    echo "===== Pushing Auth Image ====="
                    docker push \
                        ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}

                    echo "===== Pushing Streaming Image ====="
                    docker push \
                        ${ECR_REGISTRY}/streaming-stream:${IMAGE_TAG}

                    echo "===== Pushing Admin Image ====="
                    docker push \
                        ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}

                    echo "===== Pushing Chat Image ====="
                    docker push \
                        ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}

                    echo "===== Pushing Frontend Image ====="
                    docker push \
                        ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}

                    echo "===== ALL IMAGES PUSHED SUCCESSFULLY ====="
                '''
            }
        }
    }
}