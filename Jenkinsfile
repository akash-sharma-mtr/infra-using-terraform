pipeline {
agent any

```
stages {

    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Terraform Init') {
        steps {
            dir('infra') {
                bat 'terraform init'
            }
        }
    }

    stage('Terraform Validate') {
        steps {
            dir('infra') {
                bat 'terraform validate'
            }
        }
    }

    stage('Terraform Plan') {
        steps {
            dir('infra') {
                bat 'terraform plan'
            }
        }
    }

    stage('Terraform Apply') {
        steps {
            dir('infra') {
                bat 'terraform apply -auto-approve'
            }
        }
    }
}
```

}
