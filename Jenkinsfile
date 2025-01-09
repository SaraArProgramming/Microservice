pipeline {
    agent any

    environment {
        // Define environment variables for AWS credentials
        AWS_ACCESS_KEY_ID = credentials('access-key')
        AWS_SECRET_ACCESS_KEY = credentials('secret-key')
        AWS_SESSION_TOKEN = credentials('session-token')
    }

    stages {
        stage("Login to EKS") {
            steps {
                script {
                    // Export AWS credentials as environment variables
                    sh '''
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}
                        aws eks --region us-east-1 update-kubeconfig --name EKS-1
                    '''
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                script {
                    // Export AWS credentials as environment variables
                    sh '''
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}
                        kubectl apply -f deployment-service.yml
                    '''
                    sleep 60
                }
            }
        }

        stage('Verify Deployment and Get LoadBalancer URL') {
            steps {
                script {
                    // Export AWS credentials as environment variables
                    sh '''
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}
                        kubectl get all
                    '''

                    // Extract the LoadBalancer URL
                    sh '''
                        echo "LoadBalancer URL for frontend-external:"
                        kubectl get svc frontend-external -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                        echo ""
                    '''
                }
            }
        }
    }
}
