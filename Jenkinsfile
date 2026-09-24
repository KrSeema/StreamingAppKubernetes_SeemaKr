stage('Test AWS Credentials') {
    steps {
        withCredentials([
            [$class: 'AmazonWebServicesCredentialsBinding',
             credentialsId: 'StreamingApp-CI-CD_seemaKr_EKS']
        ]) {
            sh '''
                aws sts get-caller-identity
                aws ecr get-login-password --region ap-south-1 \
                  | docker login \
                  --username AWS \
                  --password-stdin 218014315198.dkr.ecr.ap-south-1.amazonaws.com
            '''
        }
    }
}