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
                withCredentials([string(credentialsId: 'BECOME_PASS_ID', variable: 'BECOME_PASS')]) {
                    sh '''
                        echo "Running Ansible Deployment..."

                        ansible-playbook -i ansible/hosts.ini ansible/deploy.yml \
                            --extra-vars "ansible_become_pass=${BECOME_PASS}"

                        echo "Playbook execution completed."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Ansible Deployment Successful!'
        }
        failure {
            echo 'Ansible Deployment Failed!'
        }
    }
}
