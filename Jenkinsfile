pipeline {
    agent any
    environment {
        AWS_CREDENTIALS_ID = 'aws'
    }
    
    stages {
        stage('Terraform Apply Test') {
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
        stage('Deploy Test Application') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        sh '''
                        aws eks update-kubeconfig --name eks-my-cluster-test --region ap-south-1
                        kubectl apply -f k8-test.yaml
                        '''
                    }
                }
            }
        }
        stage('Fetch Test Service Endpoint') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        def externalIp = sh(script: '''
                            kubectl get svc my-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                            ''', returnStdout: true).trim()
                        
                        if (externalIp) {
                            echo "Test Service External Endpoint: ${externalIp}"
                            env.ENDPOINT_URL = "http://${externalIp}:8082"
                        } else {
                            error "Failed to fetch the test service external endpoint."
                        }
                    }
                }
            }
        }
        stage('Run Selenium Test on Test Environment') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install selenium
                        python3 run.py ${ENDPOINT_URL}
                        '''
                    }
                }
            }
        }
        stage('Terraform Apply Prod') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        sh '''
                        terraform workspace select prod || terraform workspace new prod
                        terraform init
                        terraform apply -auto-approve
                        '''
                    }
                }
            }
        }
        stage('Deploy Prod Application') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        sh '''
                        aws eks update-kubeconfig --name eks-my-cluster-prod --region ap-south-1
                        kubectl apply -f k8-prod.yaml
                        '''
                    }
                }
            }
        }
        stage('Fetch Prod Service Endpoint') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        def externalIp = sh(script: '''
                            kubectl get svc my-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                            ''', returnStdout: true).trim()
                        
                        if (externalIp) {
                            echo "Prod Service External Endpoint: ${externalIp}"
                            env.ENDPOINT_URL = "http://${externalIp}:8082"
                        } else {
                            error "Failed to fetch the production service external endpoint."
                        }
                    }
                }
            }
        }
    }
}
