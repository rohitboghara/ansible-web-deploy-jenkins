pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible/ansible.cfg"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/rohitboghara/ansible-web-deploy-jenkins.git'
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh '''
                ansible-playbook -i ansible/inventory/hosts.ini ansible/playbook.yml --become
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Ansible Deployment Successful!'
        }
        failure {
            echo '❌ Ansible Deployment Failed!'
        }
    }
}

