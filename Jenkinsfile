pipeline {
    agent {
        docker {
                image 'cytopia/ansible'
            }
        }

    }
    stages {
        stage('Create a file with playbook') {
            environment {
                ANSIBLE_KEY = credentials('vm_frosty')
            }
            steps {
                sh "ansible --version"
                sh 'ansible-playbook -i hostfile --private-key=$ANSIBLE_KEY playbook.yml'
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
