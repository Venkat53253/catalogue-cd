pipeline {
    agent {
        label 'agent1'
    }
     environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '622072308398'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }
    options {
              timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    // Build
    stages {
        stage('Check Status'){
    steps{
        script{
            withAWS(credentials: 'aws-cred', region: 'us-east-1') {

                sh """
                    aws eks update-kubeconfig \
                    --region $REGION \
                    --name "$PROJECT-${params.deploy_to}"
                """

                def deploymentStatus = sh(
                    returnStdout: true,
                    script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo FAILED"
                ).trim()

            }
        }
    }
}
        
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01.namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                            kubectl apply -f application.yaml
                        """
                    }
                }
            }
        }
        
    }

    post {
        always {
            echo 'not completed'
            deleteDir()    
        }
        success {
            echo 'sucess'
        }
        failure {
            echo 'fail'
        }
    }
    
}
