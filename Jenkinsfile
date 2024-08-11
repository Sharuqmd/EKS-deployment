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
        stage('Fetch Service Endpoint') {
            steps {
                script {
                    // Fetch the service external IP
                    def externalIp = sh(script: '''
                        kubectl get svc your-service-name -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
                        ''', returnStdout: true).trim()
                    
                    // Ensure the IP is properly formatted
                    if (externalIp) {
                        echo "Service External IP: ${externalIp}"
                        // Set the endpoint URL environment variable for the Selenium script
                        env.ENDPOINT_URL = "http://${externalIp}:8080"
                    } else {
                        error "Failed to fetch the service external IP."
                    }
                }
            }
        }
        stage('Run Selenium Test') {
            steps {
                script {
                    // Run the Selenium script with the endpoint URL
                    sh "python3 run.py ${ENDPOINT_URL}"
                }
            }
        }
    }
}
