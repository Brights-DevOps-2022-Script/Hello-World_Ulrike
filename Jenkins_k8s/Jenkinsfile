pipeline {
    agent {
        docker {
            image 'gcr.io/cloud-builders/kubectl'
        }
    }
    environment {
        ACRCreds = credentials('acr_creds')
        KUBECONFIG = credentials('k8s_config')
    }
    stages {
        stage('Testing kubectl') {
            steps {
                script {
                    sh 'kubectl apply -f nginx-namespace.yaml'
                }
            }
        }
    }
}