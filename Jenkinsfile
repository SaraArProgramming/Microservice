pipeline {
    agent any

    stages {
        stage("Login to EKS") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'access-key', variable: 'AWS_ACCESS_KEY'),
                                     string(credentialsId: 'secret-key', variable: 'AWS_SECRET_KEY'),
                                     string(credentialsId: 'session-token', variable: 'AWS_SESSION_TOKEN')]) {
                        sh "aws eks --region us-east-1 update-kubeconfig --name EKS-1"
                    }
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                script {
                    // Apply the deployment and service files (default namespace)
                    sh '''
                        kubectl apply -f deployment-service.yml
                    '''
                    sleep 60
                }
            }
        }

        stage('Verify Deployment and Get LoadBalancer URL') {
            steps {
                script {
                    // Print all resources to verify the deployment
                    sh '''
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
