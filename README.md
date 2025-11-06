# Jenkins SSH & Setup Summary

This guide explains how to set up Jenkins for Ansible deployment with SSH access and password-based credentials using Secret Text.

---

## 🔧 1. Generate SSH Key (on Jenkins server)

```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@server"
```

Press **Enter** for default location (`/var/lib/jenkins/.ssh/id_rsa`) and **no passphrase**.

---

## 🚀 2. Copy SSH Key to Target Server

```bash
ssh-copy-id user@<TARGET_IP>
```

Example:

```bash
ssh-copy-id ubuntu@192.168.1.100
```

Verify connection:

```bash
ssh user@<TARGET_IP>
```

---

## 🧠 3. Add Jenkins Credentials (Global)

### ➤ Secret Text (for become/sudo password)

1. Go to **Jenkins Dashboard → Manage Jenkins → Credentials → System → Global credentials (unrestricted)**.
2. Click **Add Credentials**.
3. Set:

   * **Kind:** Secret text
   * **Secret:** your `sudo` or `become` password
   * **ID:** `BECOME_PASS_ID`
   * **Description:** `Ansible Become Password`
4. Save.

### ➤ SSH Username with Private Key (for SSH login)

1. Go to **Manage Jenkins → Credentials → Global credentials (unrestricted)**.
2. Click **Add Credentials**.
3. Set:

   * **Kind:** SSH Username with private key
   * **Username:** `user` (same as remote user)
   * **Private Key:** Choose *Enter directly* → Paste contents of `/var/lib/jenkins/.ssh/id_rsa`
   * **ID:** `SSH_KEY_ID`
   * **Description:** `Jenkins SSH Access Key`
4. Save.

---

## ⚙️ 4. Update Jenkinsfile Example

```groovy
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
                withCredentials([
                    string(credentialsId: 'BECOME_PASS_ID', variable: 'BECOME_PASS'),
                    sshUserPrivateKey(credentialsId: 'SSH_KEY_ID', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')
                ]) {
                    sh '''
                        echo "🚀 Starting Ansible Deployment..."

                        ansible-playbook -i ansible/hosts.ini ansible/deploy.yml \
                            --extra-vars "ansible_become_pass=${BECOME_PASS}" \
                            -u ${SSH_USER} --private-key=${SSH_KEY}

                        echo "✅ Deployment Completed."
                    '''
                }
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
```

---

## 📂 5. Directory Structure

```
ansible-web-deploy-jenkins/
├── ansible/
│   ├── ansible.cfg
│   ├── hosts.ini
│   └── deploy.yml
└── Jenkinsfile
```

---

## ✅ 6. Verify Setup

* Run Jenkins job.
* Jenkins uses SSH key for connecting to target host.
* Jenkins injects secret text for sudo password securely.
* No password shown in logs.

---

## 🧾 Summary

✔️ SSH key generated and copied to remote host
✔️ Jenkins credentials created (SSH + Secret Text)
✔️ Jenkinsfile updated with secure credentials
✔️ Ansible deploys automatically via Jenkins pipeline
