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
                aws ecr get-login-password --region ${AWS_REGION} |
                docker login --username AWS --password-stdin ${ECR_REGISTRY}
            '''
        }
    }
}