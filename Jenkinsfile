pipeline {
    agent any
    environment {
        AWS_REGION            = "ap-south-1"
        TF_DIR                = "terraform"
        ANSIBLE_DIR           = "ansible"
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/manisaws821-ui/todo-app.git'
            }
        }
        stage('Terraform Init') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform Validate') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform validate'
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
        stage('Wait for EC2') {
            steps {
                sh 'sleep 60'
            }
        }
        stage('Generate Inventory') {
            steps {
                script {
                    def FRONTEND_IP = sh(
                        script: "cd ${TF_DIR} && terraform output -raw frontend_ip",
                        returnStdout: true
                    ).trim()
                    def BACKEND_IP = sh(
                        script: "cd ${TF_DIR} && terraform output -raw backend_ip",
                        returnStdout: true
                    ).trim()
                    writeFile file: "${ANSIBLE_DIR}/inventory.ini", text: """
[frontend]
${FRONTEND_IP} ansible_user=ec2-user ansible_ssh_private_key_file=/home/ubuntu/todo-key.pem

[backend]
${BACKEND_IP} ansible_user=ec2-user ansible_ssh_private_key_file=/home/ubuntu/todo-key.pem
"""
                }
            }
        }
        stage('Ansible Ping') {
            steps {
                dir("${ANSIBLE_DIR}") {
                    sh 'ansible all -i inventory.ini -m ping --ssh-extra-args="-o StrictHostKeyChecking=no"'
                }
            }
        }
        stage('Deploy Application') {
            steps {
                dir("${ANSIBLE_DIR}") {
                    sh 'ansible-playbook -i inventory.ini deploy.yml --ssh-extra-args="-o StrictHostKeyChecking=no"'
                }
            }
        }
        stage('Verify') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform output'
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment successful'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}
