pipeline {
    agent any

    environment {
        APP_NAME = 'devops-webapp'
        IMAGE_TAG = '$BUILD_NUMBER'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Test & Lint') {
            steps {
                echo 'Running static sanity checks...'
                sh '''
                    test -f index.html || exit 1
                    test -f style.css || exit 1
                    grep -q "DevOps" index.html
                    echo "Static asset verification passed."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker container image...'
                sh '''
                    docker build -t devops-webapp:latest .
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to K3s cluster...'
                sh '''
                    # Import image into K3s containerd
                    docker save devops-webapp:latest | ctr -n k8s.io images import -

                    # Apply Kubernetes manifests
                    kubectl apply -f k8s/deployment.yaml

                    # Verify rollout status
                    kubectl rollout status deployment/devops-webapp --timeout=60s
                '''
            }
        }

        stage('Verify Deployment & Health') {
            steps {
                echo 'Verifying application health...'
                sh '''
                    kubectl get pods -l app=devops-webapp -o wide
                    kubectl get svc devops-webapp-service
                    curl -s -I http://localhost:30080 | grep "HTTP/"
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline Completed Successfully! Application is live on port 30080."
        }
        failure {
            echo "CI/CD Pipeline Failed. Check stage logs."
        }
    }
}