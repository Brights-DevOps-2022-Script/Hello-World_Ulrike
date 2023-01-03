pipeline {
    agent {
        docker {
                image 'cytopia/ansible'
            }
        }
        environment {
            ANSIBLE_KEY = credentials('vm_frosty')
    }
    stages {
        stage('build') {
            steps {
                sh "ansible --version"
                sh 'ansible-playbook -i --private-key=$ANSIBLE_KEY playbook.yml .txt'
            }
        }
        stage('test') {
            steps {
                sh 'echo We did it!.'
            }
        }
        stage('deploy') {
            steps {
                sh 'echo We are awesome!'
            }
        }
    }
}
