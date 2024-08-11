EKS Deployment and Selenium Testing Project
Overview

This project demonstrates the deployment of an application using Amazon EKS (Elastic Kubernetes Service) in a test environment, utilizing Terraform for cluster creation. After deploying the application, Selenium tests are executed to ensure functionality. The project then creates a production environment with the same configuration using a different Terraform workspace, followed by another round of Selenium testing.
Prerequisites

    Terraform: To manage infrastructure as code.
    AWS CLI: For interacting with AWS services.
    kubectl: For Kubernetes command-line interactions.
    Python 3: For running Selenium tests.
    Selenium: For browser automation testing.

Pipeline Overview
1. Terraform Apply Test

    Purpose: Initializes and applies Terraform configurations for the test environment.
    Commands:
        terraform workspace select test || terraform workspace new test
        terraform init
        terraform apply -auto-approve

2. Deploy Test Application

    Purpose: Configures kubectl to interact with the EKS cluster and deploys the application using k8-test.yaml.
    Commands:
        aws eks update-kubeconfig --name eks-my-cluster-test --region ap-south-1
        kubectl apply -f k8-test.yaml

3. Fetch Test Service Endpoint

    Purpose: Retrieves the external service endpoint for the deployed application in the test environment.
    Commands:
        kubectl get svc my-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

4. Run Selenium Test on Test Environment

    Purpose: Executes Selenium tests using the retrieved service endpoint.
    Commands:
        python3 -m venv venv
        . venv/bin/activate
        pip install selenium
        python3 run.py ${ENDPOINT_URL}

5. Terraform Apply Prod

    Purpose: Initializes and applies Terraform configurations for the production environment using a separate workspace.
    Commands:
        terraform workspace select prod || terraform workspace new prod
        terraform init
        terraform apply -auto-approve

6. Deploy Prod Application

    Purpose: Configures kubectl to interact with the production EKS cluster and deploys the application using k8-prod.yaml.
    Commands:
        aws eks update-kubeconfig --name eks-my-cluster-prod --region ap-south-1
        kubectl apply -f k8-prod.yaml

7. Fetch Prod Service Endpoint

    Purpose: Retrieves the external service endpoint for the deployed application in the production environment.
    Commands:
        kubectl get svc my-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
