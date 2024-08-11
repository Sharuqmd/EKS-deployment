pipeline {
    agent any
    environment {
        AWS_CREDENTIALS_ID = 'aws'
    }
    stages {
        stage('Terraform Apply') {
            steps {
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                script {
                    sh '''
                    terraform workspace select test || terraform workspace new test
                    terraform init
                    terraform apply -auto-approve
                    '''
                }
              }
            }
        }
        stage('Configure kubectl') {
            steps {
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                script {
                    sh '''
                    aws eks update-kubeconfig --name eks-my-cluster --region ap-south-1
                    '''
                    sh '''
                    kubectl apply -f k8.yaml
                    '''
                }
              }
            }
        }
        stage('Selenium test') {
            steps {
              sh " python3 -m venv my-venv "
              sh " my-venv/bin/pip install selenium "
              sh " source my-env/bin/activate"
              sh " python3 test.py"
            }
        }
    }
}

