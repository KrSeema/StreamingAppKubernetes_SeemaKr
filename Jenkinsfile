pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Source') {
            steps {
                sh '''
                    echo "===== PROJECT STRUCTURE ====="
                    pwd
                    ls -la

                    echo "===== DOCKERFILES ====="
                    find backend frontend -name Dockerfile -print

                    echo "===== HELM CHART ====="
                    ls -la streamingapp/
                '''
            }
        }
    }
}