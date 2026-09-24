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
                            docker login --username AWS --password-stdin "${ECR_REGISTRY}"
                    '''
                }
            }
        }
    }
}