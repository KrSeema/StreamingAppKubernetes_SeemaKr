pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '218014315198'
        ECR_REGISTRY = '218014315198.dkr.ecr.ap-south-1.amazonaws.com'
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

                    echo "===== Docker Images Built ====="
                    docker images | grep -E 'streaming-(auth|stream|admin|chat|frontend)'
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(
                    credentials: 'StreamingApp-CI-CD_seemaKr',
                    region: 'ap-south-1'
                ) {
                    sh '''
                        set -e

                        echo "===== AWS Identity ====="
                        aws sts get-caller-identity

                        echo "===== Logging in to Amazon ECR ====="
                        aws ecr get-login-password --region "${AWS_REGION}" |
                            docker login \
                                --username AWS \
                                --password-stdin "${ECR_REGISTRY}"
                    '''
                }
            }
        }

        stage('Push Images to ECR') {
            steps {
                withAWS(
                    credentials: 'StreamingApp-CI-CD_seemaKr',
                    region: 'ap-south-1'
                ) {
                    sh '''
                        set -e

                        echo "===== Tagging Auth Image ====="
                        docker tag \
                            streaming-auth:${IMAGE_TAG} \
                            ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}

                        echo "===== Pushing Auth Image ====="
                        docker push \
                            ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}

                        echo "===== Tagging Streaming Image ====="
                        docker tag \
                            streaming-stream:${IMAGE_TAG} \
                            ${ECR_REGISTRY}/streaming-stream:${IMAGE_TAG}

                        echo "===== Pushing Streaming Image ====="
                        docker push \
                            ${ECR_REGISTRY}/streaming-stream:${IMAGE_TAG}

                        echo "===== Tagging Admin Image ====="
                        docker tag \
                            streaming-admin:${IMAGE_TAG} \
                            ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}

                        echo "===== Pushing Admin Image ====="
                        docker push \
                            ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}

                        echo "===== Tagging Chat Image ====="
                        docker tag \
                            streaming-chat:${IMAGE_TAG} \
                            ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}

                        echo "===== Pushing Chat Image ====="
                        docker push \
                            ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}

                        echo "===== Tagging Frontend Image ====="
                        docker tag \
                            streaming-frontend:${IMAGE_TAG} \
                            ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}

                        echo "===== Pushing Frontend Image ====="
                        docker push \
                            ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}

                        echo "===== All Images Pushed Successfully ====="
                    '''
                }
            }
        }

    }

    post {
        success {
            echo 'StreamingApp CI/CD pipeline completed successfully.'

            withCredentials([
                string(
                    credentialsId: 'streamingapp-slack-webhook',
                    variable: 'SLACK_WEBHOOK_URL'
                )
            ]) {
                sh '''
                    set -e

                    curl -sS -X POST \
                        -H 'Content-Type: application/json' \
                        --data '{"text":"🚀 StreamingApp CI/CD SUCCESS"}' \
                        "${SLACK_WEBHOOK_URL}"
                '''
            }
        }

        failure {
            echo 'StreamingApp CI/CD pipeline failed.'

            withCredentials([
                string(
                    credentialsId: 'streamingapp-slack-webhook',
                    variable: 'SLACK_WEBHOOK_URL'
                )
            ]) {
                sh '''
                    set -e

                    curl -sS -X POST \
                        -H 'Content-Type: application/json' \
                        --data '{"text":"❌ StreamingApp CI/CD FAILED - Check Jenkins Console Output"}' \
                        "${SLACK_WEBHOOK_URL}"
                '''
            }
        }
    }
}