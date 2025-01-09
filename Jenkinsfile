pipeline {
    agent any

    stages {
        stage("Login to EKS") {
            steps {
                script {
                    // Directly set AWS credentials (not recommended)
                    sh """
                        export AWS_ACCESS_KEY_ID=ASIATXPNSVPQ4YMYASLU
                        export AWS_SECRET_ACCESS_KEY=FWiGpbmM5+TMO/Xn60m4rIi+iSsbxhWj21rcvESo
                        aws sts get-caller-identity
                        aws eks --region us-east-1 update-kubeconfig --name EKS-1
                    """
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                script {
                    // Apply the deployment and service files (default namespace)
                    sh """
                        kubectl apply -f deployment-service.yml
                    """
                    sleep 60
                }
            }
        }

        stage('Verify Deployment and Get LoadBalancer URL') {
            steps {
                script {
                    // Print all resources to verify the deployment
                    sh """
                        kubectl get all
                    """

                    // Extract the LoadBalancer URL
                    sh """
                        echo "LoadBalancer URL for frontend-external:"
                        kubectl get svc frontend-external -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                        echo ""
                    """
                }
            }
        }
    }
}
