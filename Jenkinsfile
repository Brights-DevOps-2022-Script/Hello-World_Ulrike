pipeline {
    agent {
        Docker{
                image 'cytopia/ansible'
            }
        }
    stages {
        stage('build') {
            steps {
                sh "ansible --version"
            }
        }
        stage('test') {
            steps {
                sh 'echo testing the pronames of the trees of the fantastic four.'
            }
        }
        stage('deploy') {
            steps {
                sh 'echo deploying this wonderfull meaningfull pipe.'
            }
        }
    }
}
