pipeline {
    agent any

    options {
        timestamps()
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

        stage('Terraform Lint') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform fmt -check -recursive' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform fmt -check -recursive' }
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform validate' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform validate' }
                    }
                }
            }
        }

        stage('Security Scan - tfsec') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''if ! command -v tfsec >/dev/null 2>&1; then echo "tfsec not installed"; exit 1; fi'''
                        dir(env.WORKING_DIR) { sh 'tfsec --no-color --format sarif --out tfsec.sarif . || true' }
                    } else {
                        bat 'where tfsec >NUL 2>&1 || (echo tfsec not installed & exit /b 1)'
                        dir(env.WORKING_DIR) { bat 'tfsec --no-color --format sarif --out tfsec.sarif . || exit /b 0' }
                    }
                }
            }
        }

        stage('Security Scan - Checkov') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''if ! command -v checkov >/dev/null 2>&1; then echo "checkov not installed"; exit 1; fi'''
                        dir(env.WORKING_DIR) { sh 'checkov -d . -o cli -o sarif --output-file-path console,checkov.sarif || true' }
                    } else {
                        bat 'where checkov >NUL 2>&1 || (echo checkov not installed & exit /b 1)'
                        dir(env.WORKING_DIR) { bat 'checkov -d . -o cli -o sarif --output-file-path console,checkov.sarif || exit /b 0' }
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform plan' }
                        // dir(env.WORKING_DIR) { sh 'terraform show -no-color tfplan > tfplan.txt' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform plan' }
                        // dir(env.WORKING_DIR) { bat 'terraform show -no-color tfplan > tfplan.txt' }
                    }
                }
            }
        }

        // stage('Manual Approval') {
        //     steps {
        //         input message: 'Approve Terraform apply to the target environment?', ok: 'Apply'
        //     }
        // }

        stage('Terraform Apply') {
            steps {
                script {
                    if (isUnix()) {
                        dir(env.WORKING_DIR) { sh 'terraform apply -auto-approve' }
                    } else {
                        dir(env.WORKING_DIR) { bat 'terraform apply -auto-approve' }
                    }
                }
            }
        }
    }
}
