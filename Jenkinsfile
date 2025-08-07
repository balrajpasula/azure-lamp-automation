pipeline {
    agent any
    environment {
        TF_VAR_subscription_id = credentials('AZURE_SUBSCRIPTION_ID')
        TF_VAR_client_id       = credentials('AZURE_CLIENT_ID')
        TF_VAR_client_secret   = credentials('AZURE_SECRET')
        TF_VAR_tenant_id       = credentials('AZURE_TENANT')
        TF_VAR_ssh_public_key  = '/path/to/your/azurekeypair.pub'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/balrajpasula/azure-lamp-automation.git'
            }
        }
        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }
        stage('Ansible Provision') {
            steps {
                script {
                    def publicIp = sh(
                        script: "terraform -chdir=terraform output -raw public_ip",
                        returnStdout: true
                    ).trim()
                    writeFile file: 'ansible/hosts', text: """
[web]
${publicIp} ansible_user=azureuser ansible_ssh_private_key_file=/home/azureuser/.ssh/id_rsa
"""
                    sh 'ansible-playbook -i ansible/hosts ansible/lamp.yml'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
