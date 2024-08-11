pipeline {
    agent any
    environment {
        AWS_CREDENTIALS_ID = 'aws'
    }
    
    stages {
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
        stage('Fetch Service Endpoint') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        // Configure kubectl
                        sh '''
                        aws eks update-kubeconfig --name eks-my-cluster --region ap-south-1
                        '''
                        
                        // Fetch the service external IP
                        def externalIp = sh(script: '''
                            kubectl get svc my-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                            ''', returnStdout: true).trim()
                        
                        // Ensure the IP is properly formatted
                        if (externalIp) {
                            echo "Service External Endpoint: ${externalIp}"
                            // Set the endpoint URL environment variable for the Selenium script
                            env.ENDPOINT_URL = "http://${externalIp}:8082"
                        } else {
                            error "Failed to fetch the service external endpoint."
                        }
                    }
                }
            }
        }
        stage('Run Selenium Test') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        // Activate Python virtual environment and run the Selenium script
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
    }
}
