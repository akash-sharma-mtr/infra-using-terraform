pipeline {
    agent any

    options {
        timestamps()
        ansiColor('xterm')
        disableConcurrentBuilds()
    }

    environment {
        WORKING_DIR = 'infra'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform init -upgrade' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform init -upgrade' }
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform fmt -check -recursive || true' }
                        dir(env.WORKING_DIR) { sh 'terraform validate' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform fmt -check -recursive || exit 0' }
                        dir(env.WORKING_DIR) { bat 'terraform validate' }
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform plan -out=tfplan' }
                        dir(env.WORKING_DIR) { sh 'terraform show -no-color tfplan > tfplan.txt' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform plan -out=tfplan' }
                        dir(env.WORKING_DIR) { bat 'terraform show -no-color tfplan > tfplan.txt' }
                    }
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: 'Approve Terraform apply to the target environment?', ok: 'Apply'
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform apply -auto-approve tfplan' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform apply -auto-approve tfplan' }
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'infra/tfplan.txt, infra/*.sarif', allowEmptyArchive: true
        }
    }
}
