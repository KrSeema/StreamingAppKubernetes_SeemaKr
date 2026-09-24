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

        stage('Test') {
            steps {
                sh '''
                    echo "Jenkinsfile syntax is valid"
                    echo "AWS Region: ${AWS_REGION}"
                    echo "ECR Registry: ${ECR_REGISTRY}"
                '''
            }
        }

    }
}