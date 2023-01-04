pipeline {
    agent {
        docker {
                image 'cytopia/ansible'
            }
        }
        environment {
            ANSIBLE_KEY = credentials('vm_frosty') // legt 2 Variablen an: //ANSIBLE_KEY_USR und //ANSIBLE_KEY_PSW, den habe ich in ansible angelegt, bei credentials, hier ID eingeben
            ANSIBLE_HOST_KEY_CHECKING = 'False' //damit der die Certificates nicht checkt und kein ERR "unreachable" ausgeworfen wird
    }
    stages {
        stage('create file') {
            steps {
                sh 'apk update'
                sh 'apk add --update --no-cache openssh sshpass'
                sh 'ansible --version'
                sh "ansible-playbook --version"
                sh "ansible-playbook -vvv -i hostfile playbook_Dockerinstall.yml -e ansible_ssh_pass=$ANSIBLE_KEY_PSW" ///-vvv debug mode, detaillierte Fehlermeldungen //Hier hole ich die Variable aus dem Environment, hab ich da angelegt.
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
