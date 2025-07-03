 pipeline {
    agent any
    environment {
        CLUSTER_NAME = "prod"
        AWS_REGION = "us-east-2"
        SONARQUBE_DOMAIN = "sonar.demosapps.com"
        HELM_CHART_VERSION = ">=8.0.0 && <= 9.0.0"  //# Check latest: https://artifacthub.io/packages/helm/sonarqube/sonarqube
    }
    stages {
        stage('Configure AWS & EKS') {
            steps {
        sh """
        
        aws sts get-caller-identity
        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
        """
    }
        }
        stage('Add Helm Repo') {
            steps {
                sh """
                helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
                helm repo update
                """
            }
        }
        stage('Deploy SonarQube') {
            steps {
                sh """
               
                helm upgrade --install sonarqube sonarqube/sonarqube \
                    --version ${HELM_CHART_VERSION} \
                    --namespace sonarqube \
                    --create-namespace \
                
                    --set persistence.enabled=true \
                    --set persistence.storageClass=gp2 \
                    --set service.type=ClusterIP \
                    -f sonarqube-values.yaml
                """
            }
        }
        stage('Configure Ingress') {
            steps {
                sh """
                kubectl apply -f sonarqube-ingress.yaml -n sonarqube
                """
            }
        }
        stage('Verify Deployment') {
            steps {
                sh """
                kubectl -n sonarqube get pods
                kubectl -n sonarqube get ingress
                echo "SonarQube will be available at https://${SONARQUBE_DOMAIN}"
                """
            }
        }
    }
}