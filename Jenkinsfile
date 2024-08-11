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
                        terraform destroy -auto-approve
                        '''
                    }
                }
            }
        }
        stage('Terraform Apply Test') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS_ID]]) {
                    script {
                        sh '''
                        terraform workspace select prod 
                        terraform init
                        terraform destory -auto-approve
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
    }
}
