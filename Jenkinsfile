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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        // Print the current Kubernetes context
                        sh 'kubectl config current-context'
                        
                        // Print the service details for debugging
                        sh 'kubectl get svc'
                        
                        // Fetch the service external DNS name
                        def externalIp = sh(script: '''
                            kubectl get svc my-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                            ''', returnStdout: true).trim()
                        
                        // Ensure the DNS name is properly formatted
                        if (externalIp) {
                            echo "Service External DNS: ${externalIp}"
                            // Set the endpoint URL environment variable for the Selenium script
                            env.ENDPOINT_URL = "http://${externalIp}:8082"
                        } else {
                            error "Failed to fetch the service external DNS name."
                        }
                    }
                }
            }
        }
        stage('Run Selenium Test') {
            steps {
                script {
                    // Ensure the ENDPOINT_URL environment variable is set
                    if (env.ENDPOINT_URL) {
                        // Run the Selenium script with the endpoint URL
                        sh "python3 run.py ${env.ENDPOINT_URL}"
                    } else {
                        error "Endpoint URL is not set. Cannot run Selenium test."
                    }
                }
            }
        }
    }
}
