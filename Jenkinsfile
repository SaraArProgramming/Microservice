pipeline {
    agent any

    environment {
        KUBECTL = '/usr/local/bin/kubectl' // Ensure kubectl is installed at this path
    }

    stages {
        stage("Login to EKS") {
            steps {
                script {
                    // Authenticate with AWS using Jenkins credentials
                    withCredentials([
                        string(credentialsId: 'access-key', variable: 'AWS_ACCESS_KEY'),
                        string(credentialsId: 'secret-key', variable: 'AWS_SECRET_KEY'),
                        string(credentialsId: 'session-token', variable: 'AWS_SESSION_TOKEN')
                    ]) {
                        // Update kubeconfig to access the EKS cluster
                        sh """
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_KEY}
                        export AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}
                        aws eks --region us-east-1 update-kubeconfig --name EKS-1
                        """
                    }
                }
            }
        }

        stage("Configure Prometheus & Grafana") {
            steps {
                script {
                    sh """
                    # Add Helm repositories
                    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts --force-update
                    helm repo update

                    # Uninstall existing release (if any)
                    helm uninstall prometheus -n prometheus || true

                    # Create namespace if it doesn't exist
                    kubectl create namespace prometheus || true

                    # Install Prometheus and Grafana using Helm
                    helm install prometheus prometheus-community/kube-prometheus-stack -n prometheus

                    # Patch services to use LoadBalancer for external access
                    kubectl patch svc prometheus-kube-prometheus-sta-prometheus -n prometheus -p '{"spec": {"type": "LoadBalancer"}}'
                    kubectl patch svc prometheus-grafana -n prometheus -p '{"spec": {"type": "LoadBalancer"}}'
                    """
                }
            }
        }

        stage("Configure ArgoCD") {
            steps {
                script {
                    sh """
                    # Create namespace if it doesn't exist
                    kubectl create namespace argocd || true

                    # Install ArgoCD using the official manifest
                    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

                    # Patch ArgoCD service to use LoadBalancer for external access
                    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
                    """
                }
            }
        }

        stage("Verify Installation") {
            steps {
                script {
                    sh """
                    # Verify Prometheus and Grafana installation
                    echo "Prometheus and Grafana Pods:"
                    kubectl get pods -n prometheus

                    echo "Prometheus and Grafana Services:"
                    kubectl get svc -n prometheus

                    # Verify ArgoCD installation
                    echo "ArgoCD Pods:"
                    kubectl get pods -n argocd

                    echo "ArgoCD Services:"
                    kubectl get svc -n argocd
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                echo "Pipeline executed successfully!"
                echo "Prometheus URL: http://$(kubectl get svc prometheus-kube-prometheus-sta-prometheus -n prometheus -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'):9090"
                echo "Grafana URL: http://$(kubectl get svc prometheus-grafana -n prometheus -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'):3000"
                echo "ArgoCD URL: http://$(kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'):80"
            }
        }
        failure {
            script {
                echo "Pipeline failed. Check the logs for errors."
            }
        }
    }
}
